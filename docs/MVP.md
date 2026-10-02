---
title: democrac.ie - MVP Scope
version: 2
last-updated: 2026-10-02
---

# democrac.ie - MVP Scope

This document defines the minimal first deliverable. Everything here is drawn from [VISION.md](VISION.md); the architecture is in [ARCHITECTURE.md](ARCHITECTURE.md). Anything not listed as MVP is out of scope until a later phase.

## MVP Feature List

### 1. Create a poll (creator, magic-link auth)

- Creator enters an email address on the create page
- The system sends a time-limited, single-use magic link through the Exodan local-mail contract
- Clicking the link sets a session cookie and lands the creator on the poll builder
- Builder: poll title, options (name plus optional description), add/remove/reorder options
- Saving produces a share URL (`/poll/{id}`) that the creator copies and distributes themselves

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

### 4. Close (manual)

- The poll creator closes the poll from their poll management view
- There is no timed or scheduled close in the MVP
- Once closed, ballots are frozen: submissions are rejected

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

- Every poll has one UUID-named directory under `STATE_DIRECTORY/polls/` containing an authoritative append-only `events.tsv`; creation, option edits, ballots, duplicate rejections, close and count events are recorded there
- Each event occupies one TSV line; JSON-encoded payloads preserve tabs and newlines in user text
- Tie draws at an elimination boundary are resolved by random draw; each draw event records the tied options in order, random-source identifier and algorithm version, sample, selection mapping and resulting elimination
- Closure freezes the accepted ballots and records the count once. Results views use that recorded count, never redraw on reload. The full count can be verified by replaying the events

## MVP Technical Commitments

- Single Go binary; stdlib HTTP; htmx + vanilla JS; one responsive CSS file covering phone, tablet and desktop
- One append-only TSV event log per UUID-named poll directory under `STATE_DIRECTORY/polls/`; no separate authoritative state or audit file
- Magic-link mail via the local-mail runtime contract only; the app never configures SMTP
- Full Exodan NixOS project-integration contract compliance (see ARCHITECTURE.md)
- Apache 2.0 licence

## Explicitly NOT in the MVP

- Voter login or email verification
- Closed-group electorates with per-voter magic links
- Changing a ballot after submission
- Timed or scheduled close
- Results visible during voting
- Multi-winner polls
- Poll purging (closed polls are retained indefinitely)
- Export, embeds, custom branding
- Any billing, tiers, or usage limits

## Acceptance Criteria

1. A first-time visitor can go from landing page to a created poll with a share URL without reading documentation
2. A voter on a phone can complete a ballot, including partial rankings, comfortably one-handed
3. Submitting a ballot, then attempting another from the same browser, shows the duplicate-deterrence message
4. Closing the poll rejects further ballots
5. After close, the results page shows every round, every elimination with its reason, every tie draw as recorded in `events.tsv`, and the winner
6. Reloading results does not recount or redraw; replaying the closed poll's event log verifies the same outcome using its recorded ballot set and draws
7. The site is usable, at full fidelity, on phone, tablet, and desktop
8. Screen-reader users can read a ballot, submit it, and read the results story

## Document Changelog

- **Version 2:** Located each poll in a UUID-named directory with one authoritative TSV event log; aligned close-time count and tie-draw evidence with the architecture.
