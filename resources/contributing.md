---
description: How to contribute to EMEI — contracts, Facilitator, and these docs.
---

# Contributing

EMEI is open source. Contributions are welcome across all three repos.

## Repos

| Repo | Stack | What lives here |
|---|---|---|
| [`EMEI-Contracts`](https://github.com/Tvarox/EMEI-Contracts) | Solidity 0.8.24, Foundry | Smart contracts and tests |
| [`EMEI-Facilitator`](https://github.com/Tvarox/EMEI-Facilitator) | Rust, Axum, alloy-rs, SQLite | HTTP server, CLI, background services |
| [`EMEI-Docs`](https://github.com/Tvarox/EMEI-Docs) | Markdown, GitBook Git Sync | This documentation site |

## Contracts

```bash
git clone https://github.com/Tvarox/EMEI-Contracts.git
cd EMEI-Contracts
forge install
forge build
forge test                  # 28 tests + 6 invariant suites
forge test -vvv             # verbose
```

Contract changes must keep the existing test suites passing and add coverage for new logic. Custom errors over `require` strings.

## Facilitator

```bash
git clone https://github.com/Tvarox/EMEI-Facilitator.git
cd EMEI-Facilitator
cp crates/emei-facilitator/.env .env
cargo build
cargo test
cargo clippy --all-targets --all-features -- -D warnings
cargo fmt --all
```

Conventions:

- Error responses must use the structured envelope. New error variants need an entry in [Revert decoding](../architecture/revert-decoding.md) and [API: Errors](../api/errors.md).
- New endpoints must be documented in `api/` and `cli/` (if surfaced via CLI).

## Docs

This site is the source of truth for `https://docs.emei.xxx` (or wherever it lives). Edit the markdown, open a PR, GitBook Git Sync picks it up.

Style:

- Diátaxis-aligned (see the [authoring prompt](../docs/AUTHORING_PROMPT.md)).
- Every page leads with the answer; details follow.
- Every code block is runnable as-is, with real addresses and env var names.
- Every parameter has a type, a constraint, and an example.
- Every error has a code, a cause, and a fix.
- Every page ends with **Next steps** or **See also** — no dead ends.

When in doubt, model new pages on the existing references for invoice / mandate / settlement.

## Reporting issues

| Type | Where |
|---|---|
| Bug in a contract | Open an issue on `EMEI-Contracts`. **Security-sensitive issues:** contact privately first. |
| Bug in the Facilitator | Open an issue on `EMEI-Facilitator` |
| Doc error | Open a PR or issue on `EMEI-Docs` |
| Feature request | Either repo, depending on layer |

## See also

- [Authoring prompt](../docs/AUTHORING_PROMPT.md) — used for generating doc content with consistency.
- [Architecture overview](../concepts/architecture.md) — start here before contributing to either repo.
