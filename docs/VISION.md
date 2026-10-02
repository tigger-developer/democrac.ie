---
title: democrac.ie - Vision
version: 4
last-updated: 2026-10-02
---

# democrac.ie

democrac.ie is a free, open-source way to make a decision together with your friends, family, or colleagues. It lets anyone create a quick poll, add a list of options, share it with others, and have them vote on their preferred choices. When the poll closes, the site explains, round by round, how the decision was reached.

## USP

- Polls use PR-STV, the voting system used in Irish elections (not to the legal letter of the Irish constitution, but in substance): it is basically instant-runoff counting. The MVP implements single-winner counting only: simple PR-STV with one winner.

- Voters rank options: first preference, second preference, third preference, and so on, as many or as few as they wish.
- A voter may rank just a single "1", or "1, 2", or "1, 2, 3, ... " all the way down the full list of options. There is no forced number of rankings and no artificial cap.
- Because this is a single-winner decision poll, no candidate reaches a quota to be elected; instead, counting proceeds as an instant runoff.
- If a voter's top choice scores low, that option is eliminated and the vote transfers to that voter's next preferred option. This continues until one option has a majority of the votes still in play.
- Also known as "instant runoff voting" or "ranked choice voting", this method ensures the winning option has broad support rather than just a simple majority.

**EVERY VOTE COUNTS:** partial rankings are fully valid. A ballot carrying only a "1" still counts in every round where that option survives; it simply becomes non-transferable (exhausted) once its final ranked option is eliminated.

## Principles

- **Free to use.** No paywalls, no vote-count limits, no feature tiers. Anyone can create a poll and anyone can vote.
- **One web app, one stylesheet.** A single responsive site that delivers a high-fidelity experience on phone, tablet, and desktop. No separate mobile site, no app.
- **Passwordless.** The site follows the same magic-link login pattern as the rest of the fleet: enter an email address, receive a time-limited login link, click, done. No passwords, no accounts to manage.
- **Explain the result, not just announce it.** Results are presented as a round-by-round story: round one tallies, the lowest-scoring option eliminated, votes transferred, next round tallies, and so on until a winner emerges - complete with a clear visualization and a plain-language explainer of what happened to each ballot.
- **Accessible.** Device friendly, screen-reader friendly, plain language.
- **Auditable.** Every count is reproducible from the recorded ballots. Ties between options at the elimination boundary are resolved by a random draw, and the draw - including what was drawn and in what order - is recorded in the poll's audit trail. Every state mutation appends to the audit trail, and the audit trail is itself plain-text CSV/TSV.

## MVP

The MVP is a single decision poll product, deliberately narrow.

1. **Create a poll:** a poll creator enters their email, verifies via magic link, names the poll, and adds options (with optional descriptions).
2. **Share:** the system generates a poll URL (`/poll/{id}`) which the creator copies and distributes by whatever channel they already use.
3. **Vote:** anyone with the link can vote anonymously. There is no login for voters. Voters enter a name in a mandatory text box (displayed with results; a self-declared label, not an identity) and rank options by assigning 1, 2, 3, ... preferences, as few or as many as they wish.
4. **Duplicate deterrence:** MVP cannot truly prevent repeat voting; it sets a cookie after voting and declines a second ballot from the same browser.
5. **Close:** the poll creator closes the poll manually. There is no scheduled close in the MVP. Closed polls and their results are retained indefinitely; the MVP performs no purging.
6. **Results:** once closed, anyone with the link sees the full round-by-round count - each round's tallies, which option was eliminated and why, how votes transferred, and the final winner - rendered as an engaging, visual, plain-language explainer.
7. **Tie handling and audit trail:** when two or more options tie for lowest score at an elimination boundary, the system eliminates one by random draw and records the draw and its order in the audit trail. Every poll mutation - creation, option edits, ballots, close, count events - appends a line to the poll's plain-text CSV/TSV audit trail.

### MVP technology

- Single Go binary, stdlib HTTP, htmx + vanilla JS, one CSS file.
- Poll state stored as append-only CSV files under `STATE_DIRECTORY` (polls, options, ballots), deduplicated on read, last write wins - the same state model as meet and upload. Counting happens in memory from the CSV state.
- Magic-link email for poll creators via the Exodan local-mail runtime contract.
- Deploys to the Exodan fleet; the app adheres to the NixOS project-integration contract (`ADDR`, `CONFIG_PATH`, `SECRETS_PATH`, `STATE_DIRECTORY`, `config/defaults.yaml`, YAML age secrets).

### What is NOT in the MVP

- Voter login or email verification (roadmap: configurable closed-group mode)
- Changing a vote after submission
- Scheduled / timed poll closure
- Live results during voting
- Multi-winner PR-STV quota counting (roadmap)
- Poll purging (roadmap: configurable; the MVP retains closed polls indefinitely)
- Export or embeds

## Roadmap

Each item is a candidate for later phases, configurable by the poll creator where noted.

- **Change vote:** creators may allow voters to revise their ballot while the poll is open.
- **Timed close:** creators may set a fixed close time instead of closing manually.
- **Live results visibility:** creators choose whether tallies are visible during voting or only after close. Default remains after close, to avoid strategic ranking.
- **Closed-group elections:** creators may supply an email list as the electorate; each participant receives a personal magic link and may cast exactly one ballot. Low friction: receive email, click, vote.
- **Duplicate detection:** heuristics and electorate restrictions to strengthen one-person-one-vote beyond the cookie.
- **Multi-winner polls (e.g. "choose the next 3 books for book club"):** full PR-STV quota counting with surplus transfers.
- **Configurable purge:** creators may set a purge-after-close period; expired polls are deleted on the fleet's usual interval-timer contract. The MVP retains everything.

LICENSE
Apache 2.0
