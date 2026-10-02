---
title: democrac.ie - Architecture
version: 3
last-updated: 2026-10-02
---

# democrac.ie - Architecture

**Status: proposed.** This document describes the intended system, not an implemented service. The first deliverable is scoped in [MVP.md](MVP.md); longer-term product direction is in [VISION.md](VISION.md).

## Overview

democrac.ie is a free ranked-choice decision poll: create a poll, add options, share a link, people rank their preferences, and when the poll closes the site explains the round-by-round instant-runoff count that produced the winner.

## Requirements

1. **Create:** a fleet operator creates the poll and its options through an application admin CLI command run by Exodan's `APP-admin` wrapper
2. **Share:** the command prints a poll URL the operator distributes through an existing channel
3. **Vote:** anyone with the link votes anonymously; no voter login. Voters enter a mandatory self-declared name and rank options 1, 2, 3, ... with as few or as many preferences as they wish
4. **Duplicate deterrence:** a cookie is set after voting and a second ballot from the same browser is declined
5. **Close:** a fleet operator closes the poll through the same admin CLI; the close operation freezes ballots and records the count
6. **Results:** after close, anyone with the link sees the round-by-round count with visualization and plain-language explanation
7. **Audit:** every poll mutation appends to the poll's authoritative TSV event log; ties at the elimination boundary are resolved by a recorded random draw

### What is NOT required

- Browser-based poll creation or management, creator login, mail, voter accounts, passwords, or voter email verification (roadmap: self-service creation and closed-group elections)
- Changing a ballot after submission (roadmap)
- Timed or scheduled close (roadmap)
- Results visible during voting (roadmap)
- Multi-winner quota counting (roadmap)
- Poll purging; the MVP retains closed polls indefinitely (roadmap: configurable purge)
- Any dependency on a third-party cloud platform

## Core Workflow

```
open -> closed
```

| State | What happens | Operator action |
|---|---|---|
| **open** | Poll and options have been created through the admin CLI; voters submit ballots and results stay hidden | Share the URL; close the poll through the admin CLI |
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

Closing a poll freezes its accepted ballot set. The close operation records the count's round tallies, eliminations, transfers, exhausted ballots, random draws and outcome in the same authoritative event stream. A results request reads the recorded outcome: it does not perform a new count or draw. Replay can independently verify the count from the recorded ballots and draw outcomes.

Each draw event records the tied option IDs in a stable order, a random-source identifier and algorithm version, the sampled value, the mapping from that value to an option, and the eliminated option. Recording the source and sample permits replay; it does not by itself prove that the source was unbiased.

## Architecture

Standalone web application as a single Go binary, following the same pattern as the fleet's other Go apps (writeback, meet, upload).

```
Go web server (single binary)
  ├── public ballot and results routing, duplicate-deterrence cookies, static files
  ├── per-poll TSV event log (append-only, authoritative)
  ├── in-memory PR-STV count
  └── operator CLI subcommands for poll creation and closure
```

No database server, no message queue, no container orchestration, no external API dependencies.

### State Model

Each poll has a directory named by its UUID beneath `STATE_DIRECTORY/polls/`. Its `events.tsv` is the sole authority for poll state, ballots, closure and the frozen count. Other poll-owned files can be added in that directory, but any derived index, snapshot or rendered asset must remain rebuildable from the event stream. This differs deliberately from meet/upload's last-write-wins CSV records: a separate state file and audit file could disagree after a partial write.

```text
STATE_DIRECTORY/
  polls/
    <poll-uuid>/
      events.tsv
      ... optional rebuildable or poll-owned files
```

Each event occupies one physical TSV line with a sequence number, timestamp, event type and JSON payload separated by tabs. JSON encoding preserves tabs and newlines in names and descriptions without letting them split the record. MVP event types cover creation, ballot submissions, rejected duplicate attempts, closure, count rounds and draw outcomes. Later option edits, if supported, would append events rather than rewrite history. The ballot event contains ordered option IDs; a replay of the log reconstructs the poll and validates the recorded result.

Writes to an individual poll are serialized. The close operation must commit the frozen count as a complete outcome before exposing results; if it is interrupted, recovery resumes from the existing events without generating a second draw. The exact durability and recovery protocol, including detection of an incomplete final record or count, remains to be specified before implementation. Corrections append new events rather than rewriting prior evidence.

Keeping all poll-owned data under one directory gives a future expiry routine a clear scope. The MVP does not purge polls; any later configurable purge must define its audit and retention consequences before removing the directory.

### Data Model

```
poll
  ├── title
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

The MVP has no browser administration, accounts or email login. Exodan's `APP-admin` wrapper runs one-off subcommands as the app user with the same runtime state and configuration as the service. Poll creation and closure are CLI-only operations; public HTTP routes cannot perform either mutation.

**Operators:** invoke the application's create and close subcommands through the wrapper. The application owns command validation and event writes; Exodan owns the host-side operator authorization and wrapper. Browser-based self-service creation, management and magic-link login are roadmap features. The MVP's poll creation command emits an open poll with its initial options; changing options after voting begins is outside its scope.

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
| **State store** | One append-only TSV event log per UUID-named poll directory under `STATE_DIRECTORY` | One authoritative history; no dual-write state/audit mismatch |
| **Counting** | In-memory Go at close | Freeze the count once; replay verifies ballots and recorded draws |
| **Frontend** | htmx + vanilla JS + one CSS file | Minimal complexity; one responsive stylesheet for phone, tablet, desktop |
| **Administration** | One-off Go CLI subcommands via Exodan `APP-admin` | No browser login or mail dependency in the MVP; host operator privileges gate commands |

## Infrastructure and Hosting

Deployment, infrastructure, and the release pipeline are managed by the separate Exodan fleet project. This section documents the application's runtime requirements only.

The app adheres to the NixOS project-integration contract:

- Runtime environment is supplied through environment variables: `ADDR`, `DEFAULT_CONFIG_PATH`, `CONFIG_PATH`, `STATE_DIRECTORY`, and `SECRETS_PATH` only if secrets are needed
- `config/defaults.yaml` declares `base_url`; the MVP does not request `local_mail`
- `config/<host>.yaml` exists per target host
- Any later required secrets use YAML under `secrets/<host>.yaml.age`; the MVP does not require a login-signing secret
- The binary binds only to `ADDR`; Caddy is the only internet-exposed process
- The Exodan-generated `APP-admin` wrapper invokes application-owned create and close subcommands with the application identity and runtime contract; the exact wrapper name follows the deployed app identity
- The Makefile has local dev targets only (`build`, `test`, `lint`, `install`)
- No port choice, systemd units, timers, DNS, or host-side files are owned by this repo

Future browser-based creator access may use Exodan's optional local-mail capability. It is not needed for the MVP CLI workflow.

Backup, process supervision, health checks, and TLS are owned by the Exodan fleet.

## Document Changelog

- **Version 3:** Replaced MVP creator login and browser management with Exodan-style admin CLI creation and closure; deferred mail and creator authentication. Created polls open with their initial options.
- **Version 2:** Replaced separate last-write-wins CSV state and audit files with one authoritative TSV event stream per UUID-named poll directory; specified count freeze on close and recorded random-source evidence. Interrupted-close recovery remains an explicit design question.
