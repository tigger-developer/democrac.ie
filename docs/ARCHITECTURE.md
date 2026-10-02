---
title: democrac.ie - Architecture
version: 1
last-updated: 2026-10-02
---

# democrac.ie - Architecture

## Overview

democrac.ie is a free ranked-choice decision poll: create a poll, add options, share a link, people rank their preferences, and when the poll closes the site explains the round-by-round instant-runoff count that produced the winner.

## Requirements

1. **Create:** a poll creator enters their email, verifies via magic link, names the poll, and adds options with optional descriptions
2. **Share:** the system generates a poll URL the creator distributes through whatever channel they already use
3. **Vote:** anyone with the link votes anonymously; no voter login. Voters enter a mandatory self-declared name and rank options 1, 2, 3, ... with as few or as many preferences as they wish
4. **Duplicate deterrence:** a cookie is set after voting and a second ballot from the same browser is declined
5. **Close:** the creator closes the poll manually
6. **Results:** after close, anyone with the link sees the round-by-round count with visualization and plain-language explanation
7. **Audit:** every poll mutation appends to a plain-text CSV/TSV audit trail; ties at the elimination boundary are resolved by a recorded random draw

### What is NOT required

- Voter accounts, passwords, or voter email verification (roadmap: closed-group elections)
- Changing a ballot after submission (roadmap)
- Timed or scheduled close (roadmap)
- Results visible during voting (roadmap)
- Multi-winner quota counting (roadmap)
- Poll purging; the MVP retains closed polls indefinitely (roadmap: configurable purge)
- Any dependency on a third-party cloud platform

## Core Workflow

```
created -> open -> closed
```

| State | What happens | Creator action |
|---|---|---|
| **created** | Poll and options defined; share URL available | Edit options, copy share URL |
| **open** | Voters submit ballots; results hidden | Close the poll |
| **closed** | Ballots frozen; round-by-round results visible to anyone with the link | - |

## Counting Algorithm

Single-winner instant runoff (PR-STV in substance, one winner):

1. Count first preferences for each option
2. If one option has a strict majority of non-exhausted ballots, it wins
3. Otherwise eliminate the option with the fewest votes; each ballot whose top surviving preference was the eliminated option transfers to its next surviving preference
4. Ballots with no surviving ranked preference are exhausted
5. Repeat until one option has a majority of the votes still in play
6. If the final two options tie, resolve by random draw

**Tie handling:** when options tie for fewest votes at an elimination boundary, one is eliminated by random draw. The draw, the options involved, and the order in which they were drawn are recorded in the audit trail.

**Exhausted ballots:** a ballot ranking only a "1" is valid and counts in every round its option survives; once its last ranked option is eliminated it is exhausted and counts in no further tally (but remains in the audit record).

The count is deterministic given the ballots and the recorded draw outcomes; it is recomputed in memory from CSV state at results-render time and is reproducible from the audit trail alone.

## Architecture

Standalone web application as a single Go binary, following the same pattern as the fleet's other Go apps (writeback, meet, upload).

```
Go web server (single binary)
  ├── routing, magic-link auth, session cookies, static files
  ├── CSV/TSV state store (append-only, last write wins)
  ├── in-memory PR-STV count
  └── sendmail adapter (subprocess) - magic-link mail via local-mail contract
```

No database server, no message queue, no container orchestration, no external API dependencies.

### State Model

All persistent state lives in `STATE_DIRECTORY` as append-only plain-text CSV files, deduplicated on read with last-write-wins semantics - the same model as meet and upload. Counting happens in memory from the CSV state.

```
polls.csv        poll_id, title, creator_email, state, created_at, closed_at
options.csv     poll_id, option_id, name, description, position, created_at
ballots.csv      poll_id, ballot_id, voter_name, rankings, submitted_at
audit.csv        poll_id, event_id, timestamp, event_type, payload
```

The audit file records every poll mutation: creation, option edits, ballot submissions, duplicate-deterrence rejections, close, and each count event (round tallies, eliminations, draw outcomes including draw order). `rankings` on a ballot is an ordered list of option IDs; the audit trail stores the same ballot content, so the entire count is reproducible from `audit.csv` alone.

A single CSV file locking discipline serializes appends; the reader tolerates a torn final line. Keys are immutable; corrections append new rows that supersede by key on read.

### Data Model

```
poll
  ├── title
  ├── creator_email (passwordless - authenticates via magic link)
  ├── state: created | open | closed
  ├── options[]
  │     ├── name
  │     ├── description (optional)
  │     └── position
  └── ballots[]
        ├── voter_name (mandatory, self-declared label, not an identity)
        └── rankings (ordered option IDs, 1..N, any length up to N)
```

### Access Model

No passwords. No accounts. Authentication is a magic email link for poll creators only.

**Creators:**
1. Enter email on the create-poll page
2. Receive a magic login link (time-limited, single-use), sent through the Exodan local-mail contract
3. Click link -> session cookie
4. Create and manage polls; close a poll

**Voters:** no authentication at all. Possession of the poll URL is the ballot access credential. After a ballot is submitted, a cookie marks the browser as having voted and further ballots from that browser are declined. The name text box is a self-declared label for display with results, not an identity or access token.

### Results Experience

After close, the results page presents the count as a story, in the site's single responsive stylesheet:

- Round-by-round tallies, one round at a time, visualized
- Which option was eliminated each round and why (fewest votes, or random draw with the recorded outcome)
- How votes transferred, in plain language
- Exhausted-ballot counts shown where relevant
- Final winner with its majority share

The rendering is server-side HTML plus progressive enhancement (vanilla JS): the full story is readable without scripting, and the round-by-round animation is layered on top. Explainer copy is written for a general audience.

## Technology Stack

| Component | Technology | Rationale |
|---|---|---|
| **Web framework** | Go (single binary) | Fast, low memory, stdlib handles HTTP/crypto; same pattern as the fleet's other apps |
| **State store** | Append-only CSV/TSV under `STATE_DIRECTORY` | Zero configuration; human-readable; audit trail is the same medium as the state store; last-write-wins on read, as in meet and upload |
| **Counting** | In-memory Go | Pure arithmetic; deterministic given ballots and recorded draws |
| **Frontend** | htmx + vanilla JS + one CSS file | Minimal complexity; one responsive stylesheet for phone, tablet, desktop |
| **Auth** | Passwordless magic email links (creators only) | No passwords; email is the sole authentication factor; session cookies for persistence |
| **Token signing** | Go `crypto/hmac` + `crypto/sha256` | Time-limited signed tokens for magic links; no external dependency |
| **Email** | Exodan local-mail contract (`SENDMAIL_PATH`) | The app never configures SMTP or holds provider credentials |

## Infrastructure and Hosting

Deployment, infrastructure, and the release pipeline are managed by the separate Exodan fleet project. This section documents the application's runtime requirements only.

The app adheres to the NixOS project-integration contract:

- Runtime environment is supplied entirely through environment variables: `ADDR`, `DEFAULT_CONFIG_PATH`, `CONFIG_PATH`, `SECRETS_PATH`, `STATE_DIRECTORY`, and (with the local-mail capability) `SENDMAIL_PATH`, `MAIL_DEFAULT_SENDER_PATH`, `MAIL_DEFAULT_SENDER_DOMAIN`
- `config/defaults.yaml` declares `base_url` and `service.capabilities.local_mail: true`
- `config/<host>.yaml` exists per target host
- Secrets are YAML under `secrets/<host>.yaml.age`
- The binary binds only to `ADDR`; Caddy is the only internet-exposed process
- A `democrac-admin` CLI wrapper is exposed for one-off operations (e.g. generating a creator login link)
- The Makefile has local dev targets only (`build`, `test`, `lint`, `install`)
- No port choice, systemd units, timers, DNS, or host-side files are owned by this repo

Mail requirements are modest: magic-link messages to poll creators only. The local-mail adapter's 10 MiB message limit and 50-recipient cap are irrelevant at this volume; the app builds complete RFC messages and owns the From header, using the sender patterns the mail contract prescribes.

Backup, process supervision, health checks, and TLS are owned by the Exodan fleet.
