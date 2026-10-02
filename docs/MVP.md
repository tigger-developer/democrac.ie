---
title: democrac.ie - MVP Scope
version: 3
last-updated: 2026-10-02
---

# democrac.ie - MVP Scope

This document defines the minimal first deliverable. Everything here is drawn from [VISION.md](VISION.md); the architecture is in [ARCHITECTURE.md](ARCHITECTURE.md). Anything not listed as MVP is out of scope until a later phase.

## MVP Feature List

### 1. Create a poll (operator CLI)

- A fleet operator invokes a one-off poll-creation subcommand through Exodan's `APP-admin` wrapper, running as the application user with the service's runtime state and configuration
- The command accepts a poll title and options (names and optional descriptions), validates them, creates an open poll with its initial options, and prints its share URL (`/poll/{id}`)
- The operator distributes the URL through an existing channel; there is no public creation page or creator login in the MVP

### 2. Vote (anonymous, link-based)

- Anyone with the poll URL can vote; there is no voter login
- Voter enters a name in a mandatory text box - a self-declared label displayed with results, not an identity
- Voter ranks options with preferences 1, 2, 3, ...; any number of preferences from one up to all options is valid
- Partial rankings are valid ballots; there is no forced number of rankings
- After submission, the voter sees a confirmation that their ballot was recorded

### 3. Duplicate deterrence (cookie only)

- After a ballot is submitted, a cookie marks the browser
- Further ballots from the same browser are declined with a plain explanation
- This is deterrence, not enforcement; it cannot stop a determined repeat voter. The MVP makes no stronger claim

### 4. Close (manual operator CLI)

- A fleet operator invokes a one-off poll-close subcommand through the same `APP-admin` wrapper, identifying the poll to close
- There is no timed or scheduled close in the MVP
- Closing freezes the accepted ballots and records the count; further submissions are rejected

### 5. Results (after close)

- Anyone with the link sees the full round-by-round count:
  - each round's tallies, visualized
  - which option was eliminated each round, and why (fewest votes, or random draw)
  - how votes transferred, in plain language
  - exhausted ballots where relevant
  - the final winner with its majority share
- Results are hidden while the poll is open
- The page explains the count to a general audience: it tells the story of the decision, not just the totals

### 6. Audit trail (always on)

- Every poll has one UUID-named directory under `STATE_DIRECTORY/polls/` containing an authoritative append-only `events.tsv`; creation, ballots, duplicate rejections, close and count events are recorded there
- Each event occupies one TSV line; JSON-encoded payloads preserve tabs and newlines in user text
- Tie draws at an elimination boundary are resolved by random draw; each draw event records the tied options in order, random-source identifier and algorithm version, sample, selection mapping and resulting elimination
- Closure freezes the accepted ballots and records the count once. Results views use that recorded count, never redraw on reload. The full count can be verified by replaying the events

## MVP Technical Commitments

- Single Go binary; stdlib HTTP; htmx + vanilla JS; one responsive CSS file covering phone, tablet and desktop
- One append-only TSV event log per UUID-named poll directory under `STATE_DIRECTORY/polls/`; no separate authoritative state or audit file
- One-off create and close commands run as the app user via Exodan's `APP-admin` wrapper; the MVP has no mail dependency or browser-based administration
- Full Exodan NixOS project-integration contract compliance (see ARCHITECTURE.md)
- Apache 2.0 licence

## Explicitly NOT in the MVP

- Voter login or email verification
- Browser-based poll creation, creator login, magic-link mail and a poll-management interface
- Editing options after poll creation
- Closed-group electorates with per-voter magic links
- Changing a ballot after submission
- Timed or scheduled close
- Results visible during voting
- Multi-winner polls
- Poll purging (closed polls are retained indefinitely)
- Export, embeds, custom branding
- Any billing, tiers, or usage limits

## Acceptance Criteria

1. A fleet operator can create a poll through the Exodan-style admin CLI and receive a share URL; an unauthenticated visitor cannot create or close a poll over HTTP
2. A voter on a phone can complete a ballot, including partial rankings, comfortably one-handed
3. Submitting a ballot, then attempting another from the same browser, shows the duplicate-deterrence message
4. Closing the poll through the admin CLI rejects further ballots and exposes its frozen result
5. After close, the results page shows every round, every elimination with its reason, every tie draw as recorded in `events.tsv`, and the winner
6. Reloading results does not recount or redraw; replaying the closed poll's event log verifies the same outcome using its recorded ballot set and draws
7. The site is usable, at full fidelity, on phone, tablet, and desktop
8. Screen-reader users can read a ballot, submit it, and read the results story

## Document Changelog

- **Version 3:** Moved MVP poll creation and manual closure to the Exodan-style operator CLI; deferred creator login, browser administration and mail.
- **Version 2:** Located each poll in a UUID-named directory with one authoritative TSV event log; aligned close-time count and tie-draw evidence with the architecture.
