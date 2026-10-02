# democrac.ie

democrac.ie is a free, open-source way to make a decision together with your friends, family, or colleagues. Create a poll, add options, share the link, and everyone ranks their preferences - 1, 2, 3, and so on, as many or as few as they like. When the poll closes, the site explains, round by round, how the decision was reached.

Polls are counted with PR-STV (the system used in Irish elections, in substance): an instant runoff where low-scoring options are eliminated and votes transfer to the next preference until one option has broad support. Every vote counts - a ballot with just a "1" is a valid ballot.

## Features

- **Free and unlimited** - no paywalls, no vote limits, no tiers
- **No accounts** - poll creators log in with a one-time email link; voters just need the poll URL
- **Explain the result, not just announce it** - round-by-round tallies, eliminations, transfers, and a plain-language story of how the winner won
- **Auditable** - every poll mutation is recorded in a plain-text CSV/TSV audit trail; the whole count is reproducible from it
- **Accessible and responsive** - one web app, one stylesheet, high fidelity on phone, tablet, and desktop

## Documentation

- [Vision](docs/VISION.md) - what democrac.ie is and where it's going
- [Architecture](docs/ARCHITECTURE.md) - system design, counting algorithm, state model
- [MVP scope](docs/MVP.md) - the minimal first deliverable and its acceptance criteria

## Development

```bash
make build
make test
make lint
```

Local development targets only; deployment is handled by the Exodan fleet's `deploy-nix` from the operator's machine.

## Licence

Apache 2.0
