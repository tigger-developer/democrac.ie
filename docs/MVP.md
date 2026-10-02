---
title: democrac.ie - MVP Scope
version: 1
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

- Every poll mutation appends a line to the poll's plain-text CSV/TSV audit trail: creation, option edits, ballots, duplicate rejections, close, and count events
- Tie draws at an elimination boundary are resolved by random draw, and the draw, the options involved, and the draw order are recorded
- The full count is reproducible from the audit trail alone

## MVP Technical Commitments

- Single Go binary; stdlib HTTP; htmx + vanilla JS; one responsive CSS file covering phone, tablet and desktop
- Append-only CSV state under `STATE_DIRECTORY`, deduplicated on read, last write wins (meet/upload model)
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
5. After close, the results page shows every round, every elimination with its reason, every tie draw as recorded in the audit trail, and the winner
6. Deleting the in-memory count and re-rendering results from the CSV state (including audit-recorded draws) reproduces the identical result
7. The site is usable, at full fidelity, on phone, tablet, and desktop
8. Screen-reader users can read a ballot, submit it, and read the results story
