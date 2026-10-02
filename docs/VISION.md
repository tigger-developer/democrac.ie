---
title: democrac.ie - Vision
version: 7
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
- **Passwordless.** When self-service poll creation arrives, it follows the fleet's magic-link login pattern: enter an email address, receive a time-limited login link, click, done. The initial operator-only creation workflow needs no browser login.
- **Explain the result, not just announce it.** Results are presented as a round-by-round story: round one tallies, the lowest-scoring option eliminated, votes transferred, next round tallies, and so on until a winner emerges - complete with a clear visualization and a plain-language explainer of what happened to each ballot.
- **Accessible.** Device friendly, screen-reader friendly, plain language.
- **Auditable.** Every count is reproducible from the recorded ballots. Ties between options at the elimination boundary are resolved by a random draw, and the draw - including what was drawn and in what order - is recorded in the poll's audit trail. Every state mutation appends to the audit trail, and the audit trail is plain-text TSV.

## MVP

The first deliverable is a single decision poll product, deliberately narrow. Its full scope, technical commitments, exclusions, and acceptance criteria are defined in [MVP.md](MVP.md).

## Roadmap

Each item is a candidate for later phases, configurable by the poll creator where noted.

- **Self-service creation and administration:** a browser poll builder and creator management view, with passwordless magic-link login and email delivery; creation and closure need not remain operator tasks.
- **Change vote:** creators may allow voters to revise their ballot while the poll is open.
- **Timed close:** creators may set a fixed close time instead of closing manually.
- **Live results visibility:** creators choose whether tallies are visible during voting or only after close. Default remains after close, to avoid strategic ranking.
- **Closed-group elections:** creators may supply an email list as the electorate; each participant receives a personal magic link and may cast exactly one ballot. Low friction: receive email, click, vote.
- **Duplicate detection:** heuristics and electorate restrictions to strengthen one-person-one-vote beyond the cookie.
- **Multi-winner polls (e.g. "choose the next 3 books for book club"):** full PR-STV quota counting with surplus transfers.
- **Configurable purge:** creators may set a purge-after-close period; expired polls are deleted on the fleet's usual interval-timer contract. The MVP retains everything.

LICENSE
Apache 2.0

## Document Changelog

- **Version 7:** Deferred browser poll creation and passwordless creator access to the roadmap; the MVP uses an operator CLI.
- **Version 6:** Clarified that the poll audit trail uses TSV; the per-poll storage design belongs in the architecture.
