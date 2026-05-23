---
description: Frequently asked questions about EMEI — design rationale, edge cases, and operational details.
---

# FAQ

## Why mUSD as the settlement asset?

mUSD is the Mantle ecosystem's yield-bearing rebasing stablecoin. EMEI uses it as the settlement asset for two reasons:

1. **Yield by default.** Settled funds earn yield from the moment they hit the vault, with no further action from the payee.
2. **Single-asset settlement surface.** Issuers don't have to track multiple stablecoins. USDC payers are auto-swapped at settlement time with a 1% slippage cap.

If you need to hold a non-rebasing asset, withdraw your vault balance after each settlement.

## What happens if my reputation drops mid-flow?

Reputation is checked twice: at `createInvoice` (both sides) and at `pay` / `collect` (payer only). If the payer's score has dropped below `minReputation` between those two moments, the payment call reverts with `ReputationTooLow` and the invoice **stays in its current status** — it is *not* auto-rejected.

The issuer can either wait for the score to recover or call `reject(invoiceId, "rep below threshold")` to terminate the invoice. → [Reputation gate](../concepts/reputation.md).

## Can a mandate be paused without revoking?

No. `revokeMandate` is terminal — once revoked, a mandate cannot be re-activated. To "pause", revoke and re-create when needed. If your application needs a soft-pause, gate the trigger logic in your own off-chain code.

## Is the Facilitator trustless?

The Facilitator is **not custodial** — it never holds user funds. It signs operator-level transactions (auto-collect, overdue scan, receipt batching) using its own keys, which need MNT for gas only.

For the trust boundaries that matter:

- **Settlement:** trustless. `EMEISettlement.settle` is callable only by `EMEIInvoice`, and the contract logic is on-chain.
- **Receipt verification:** trustless. Anyone can call `EMEIReceipt.verifyInclusion` directly.
- **Mandate validation:** trustless. `validateAndDecrement` is enforced on-chain.
- **HTTP key handling:** **you trust the Facilitator operator** with the `X-Private-Key` header. The Facilitator is intended for self-hosting, not as a shared service.

→ [Security model](../concepts/security.md).

## Why is `collect` permissionless?

Because the **mandate is the access control**. The mandate already encodes which issuers can bill, which categories are allowed, and how much can be spent. The `collect` function just enforces that scope. Allowing anyone to call it means a third-party indexer, the Facilitator's Auto-Collector, or even the issuer themselves can trigger the on-due-date payment without coordinating signatures.

This is by design and is what makes mandate auto-collection feel like direct-debit.

## What's the difference between `pay` and `collect`?

| | `pay(invoiceId)` | `collect(invoiceId, mandateId)` |
|---|---|---|
| Caller | Payer only | Anyone |
| Auth source | `msg.sender == payer` | Mandate scope |
| Used for | `PAY_LINK` mode | `MANDATE` mode |
| Wallet sigs | 2 (approve + pay) for browser; 1 if approval is pre-set | 0 from payer |

## Can I use real USDC instead of MockUSDC?

Not on the current testnet deployment. `EMEISettlement` is configured against `MockUSDC` at a known address. When mainnet ships, the real USDC contract will be wired in.

## Does EMEI support recurring subscriptions?

Yes — through mandates. A subscription is just a series of invoices in `MANDATE` mode where each invoice's amount fits within the mandate's `remainingCap`. The issuer issues + presents on each cycle; the Auto-Collector settles automatically. When the cap runs out (`status: EXHAUSTED`), the payer creates a new mandate.

Native "recurring" semantics with auto-renewal are intentionally **not** part of the protocol — explicit mandate creation is the security boundary.

## What happens to an OVERDUE invoice?

`OVERDUE` is **not** terminal. The payer (or the Auto-Collector, if a mandate exists) can still pay it. It just signals "the due date has passed". The Overdue Scanner background service flips invoices into this status permissionlessly once `block.timestamp > dueDate`.

The only terminal outcomes are `PAID` and `REJECTED`. → [The invoice lifecycle](../concepts/invoice-lifecycle.md).

## How do receipts get anchored?

Every paid invoice produces a 32-byte leaf. The Receipt Batcher background service runs every 30 seconds: it groups unbatched leaves into a Merkle tree, computes the root, and calls `EMEIReceipt.postMerkleRoot(batchN, root)`. Anyone can later call `EMEIReceipt.verifyInclusion(batch, leaf, proof)` to verify a receipt was anchored — without trusting the Facilitator.

→ [Receipt anchoring](../concepts/receipts.md).

## Will the API change before mainnet?

The HTTP/CLI surface is stable across the testnet → mainnet transition. Contract addresses will change (new deployments) and `MockERC8004` / `MockmUSD` / `MockUSDC` will be replaced by their production counterparts, but you should not need to rewrite client code beyond updating env vars.

## See also

- [Glossary](../welcome/glossary.md)
- [Troubleshooting](troubleshooting.md)
- [Security model](../concepts/security.md)
