---
description: Notable changes to EMEI contracts, Facilitator, and CLI.
---

# Changelog

This changelog tracks user-visible changes. For commit-level history, see the source repos:

- [`EMEI-Contracts` releases](https://github.com/Tvarox/EMEI-Contracts/releases)
- [`EMEI-Facilitator` releases](https://github.com/Tvarox/EMEI-Facilitator/releases)

## Unreleased

- Documentation site bootstrapped on GitBook.

## v0.1 — Mantle Sepolia testnet

Initial public testnet deployment.

**Contracts (Mantle Sepolia, chain ID 5003):**

- `EMEIInvoice` at `0xC35f709255D7199394655F16008e8d1A3AD80005` — full state machine with `ISSUED → PRESENTED → PAID/OVERDUE/REJECTED` transitions.
- `EMEIMandate` at `0xF48C3bd4FE046629A9c12A39693f39c297893bD8` — scoped pre-authorization with cap, counterparty list (≤50), category list (≤20), and validity window.
- `EMEISettlement` at `0xfdCb7bA077069A7Da44711Ee6bdB49174AFA4dD0` — direct mUSD transfer + USDC→mUSD swap with 1% slippage cap, `MUSD_REBASE` and `SUSDE` vaults.
- `Bay8004` at `0xE61B57D84fb55E2601ab47B83c367612E348d409` — reputation adapter, weighted scoring (default 30/20/50 split).
- `EMEIReceipt` at `0x558a20766d5998765B056597b8b78fe1914f3969` — Merkle root anchoring with on-chain `verifyInclusion`.

**Mocks:**

- `MockERC8004` at `0x4B560970423B08632bC2Aa31D0a70e29e66Fca37`
- `MockmUSD` at `0xb4C74657Ef45AA95E91BBac1db7f9C964D1cAeAD`
- `MockUSDC` at `0x2a2f576b67773d4CFe6157C3150BbAB8fFb0FDF6`

**Facilitator:**

- 14 HTTP endpoints (8 ✅ auth, 6 ❌ public).
- 4 background services: Auto-Collector (10s), Overdue Scanner (60s), Receipt Batcher (30s), Event Indexer (continuous).
- SQLite event index with cursor-paginated `/emei/statement`.
- Docker image with multi-stage build.

**CLI (`emei`):**

- All 14 endpoints surfaced as commands.
- JSON output for agent runtimes.

## See also

- [Upgrading & versioning](../ops/upgrading.md)
- [Deployed addresses](../contracts/addresses.md)
