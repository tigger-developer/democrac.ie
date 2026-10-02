---
title: democrac.ie
version: 2
last-updated: 2026-10-02
---

# democrac.ie

democrac.ie is a free, open-source way to make a decision together with your friends, family, or colleagues. Create a poll, add options, share the link, and everyone ranks their preferences - 1, 2, 3, and so on, as many or as few as they like. When the poll closes, the site explains, round by round, how the decision was reached.

Polls are counted with PR-STV (the system used in Irish elections, in substance): an instant runoff where low-scoring options are eliminated and votes transfer to the next preference until one option has broad support. Every vote counts - a ballot with just a "1" is a valid ballot.

## Features

- **Free and unlimited** - no paywalls, no vote limits, no tiers
- **No voter accounts** - voters just need the poll URL; in the MVP, a fleet operator creates and closes polls using an admin CLI, with self-service creation planned later
- **Explain the result, not just announce it** - round-by-round tallies, eliminations, transfers, and a plain-language story of how the winner won
- **Auditable** - every poll mutation is recorded in its append-only TSV event log; the whole count is reproducible from it
- **Accessible and responsive** - one web app, one stylesheet, high fidelity on phone, tablet, and desktop

## Documentation

- [Vision](docs/VISION.md) - what democrac.ie is and where it's going
- [Architecture](docs/ARCHITECTURE.md) - system design, counting algorithm, state model
- [MVP scope](docs/MVP.md) - the minimal first deliverable and its acceptance criteria

## Development

The application and its Make targets have not yet been implemented. Deployment is intended to follow the Exodan fleet's `deploy-nix` contract from the operator's machine; poll creation and closure in the MVP use its one-off admin CLI wrapper.

## Licence

Apache 2.0

## Document Changelog

- **Version 2:** Distinguished MVP operator-created polls from planned passwordless self-service creation.
