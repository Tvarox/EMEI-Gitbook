---
description: The case for programmable invoicing — why agents and humans need an on-chain billing rail with scoped pre-authorization, reputation gating, and yield by default.
---

# Why programmable invoicing?

Autonomous agents transact constantly: API calls, compute, data, micro-services, recurring subscriptions. Today they pay using primitives that were built for humans clicking buttons:

- **Card rails** — chargebacks, KYC friction, no machine-native API for issuance.
- **Bank transfers** — slow, jurisdictional, no programmability.
- **Raw on-chain transfers** — no invoice, no terms, no audit trail, no scope-limited pre-authorization.

EMEI fills the gap with a primitive built for the agent economy.

## What "programmable invoicing" means

An invoice on EMEI is a **first-class on-chain object** that carries:

- **Identity on both sides** (ERC-8004 registry).
- **Reputation gating** at creation *and* at payment (Bay8004).
- **Terms** — due-on-receipt, Net-N (1–365 days), or up to 10 milestones.
- **Line items** with categories (max 50 items).
- **A collection mode** — pre-authorized mandate or pay-link.
- **A status machine** — ISSUED → PRESENTED → PAID, with terminal REJECTED and reversible OVERDUE.
- **A settlement proof** — cryptographic receipt anchored in a Merkle batch on-chain.

You don't build any of this yourself. You call `createInvoice` and the protocol handles the rest.

## The four properties that matter

### 1. Scoped pre-authorization (mandates)

A payer can authorize *categories of future spend* without authorizing arbitrary withdrawals. A mandate carries:

- A **spend cap** (e.g. 1,000 mUSD over the period).
- An **allowed counterparty list** (up to 50).
- An **allowed category list** (up to 20).
- A **validity window** (`validFrom`, `validUntil`).

An invoice that matches the scope settles automatically. An invoice that doesn't is rejected at the contract level — no value moves. Think direct-debit, but the bank is a contract you can read and audit.

### 2. Reputation gating on both sides

Bad actors can't grief the rail. Both issuer and payer must score above a threshold at invoice creation; the payer is re-checked at payment. Score is computed as a weighted blend (transaction-size 30%, time-decay 20%, category 50%) over feedback from completed settlements. → [Reputation gate](../concepts/reputation.md)

### 3. Yield by default

Settled mUSD is auto-routed to a rebasing vault (default `MUSD_REBASE`, optional `SUSDE`). Idle balances earn yield from the moment they arrive. Withdrawals are explicit — the payee opts in to "stop earning" rather than opting in to "start earning". → [Settlement & yield](../concepts/settlement.md)

### 4. Cryptographic receipts

Every paid invoice produces a receipt. Receipts are batched every ~30 seconds and the Merkle root is posted on-chain by the Receipt Batcher service. Anyone can verify a receipt's inclusion via `verifyInclusion` — no trust in the facilitator required for proof of payment. → [Receipt anchoring](../concepts/receipts.md)

## Who EMEI is for

- **Agent developers** building services that bill other agents or humans.
- **Platforms** that orchestrate agents and need a settlement primitive with audit trail.
- **Researchers** studying machine-economic infrastructure.

## Who EMEI is *not* for (yet)

- High-throughput consumer micropayments — the rail is optimized for invoices, not streams.
- Use cases that require permissionless mainnet today — EMEI is currently testnet-only on Mantle Sepolia.

## Next steps

- [How EMEI compares](comparison.md) — vs. x402, ERC-7710, traditional invoicing.
- [Quickstart](../getting-started/quickstart.md) — issue your first invoice.
- [Core concepts](../concepts/architecture.md) — read the design.

## See also

- [Glossary](glossary.md)
- [FAQ](../resources/faq.md)
