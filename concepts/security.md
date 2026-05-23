---
description: EMEI's security model — trust boundaries, contract authorization rules, and known limitations.
---

# Security model

This page summarizes EMEI's security posture. For specific contract authorization, see each contract's reference page.

## Trust boundaries

| Component | Trust assumption |
|---|---|
| Contracts on Mantle Sepolia | Trust-minimized. Anyone can interact directly. |
| Facilitator HTTP server | Trust the operator (typically *you*). Self-host. |
| Operator and batcher private keys | Hold MNT only — never custody user funds. |
| Bay8004 / ERC-8004 registry | Trust the registry implementation. On testnet this is `MockERC8004`. |
| Pay-link calldata | Verifiable — clients can re-derive locally. |

The Facilitator is **not custodial**. It signs operator-level transactions (auto-collect, overdue scan, receipt batching) but never moves user funds beyond what the contracts authorize.

## Contract-level access control

| Function | Who can call |
|---|---|
| `EMEIInvoice.createInvoice` | Anyone |
| `EMEIInvoice.present` | Issuer only |
| `EMEIInvoice.pay` | Payer only |
| `EMEIInvoice.collect` | **Permissionless** — bounded by mandate scope |
| `EMEIInvoice.markOverdue` | **Permissionless** — bounded by `block.timestamp` |
| `EMEIInvoice.reject` | Issuer only |
| `EMEIInvoice.setMinReputation` | Owner only |
| `EMEIMandate.validateAndDecrement` | `EMEIInvoice` only |
| `EMEIMandate.revokeMandate` | Mandate's payer only |
| `EMEISettlement.settle` | `EMEIInvoice` only |
| `EMEISettlement.withdraw` | Payee only (their own balance) |
| `EMEIReceipt.postMerkleRoot` | Authorized poster only |

## Defensive properties

- **CEI pattern.** All contracts follow Checks–Effects–Interactions to mitigate reentrancy.
- **Custom errors.** All revert paths use named custom errors (gas-efficient, structured).
- **Terminal states.** `PAID` and `REJECTED` cannot transition further.
- **Slippage cap.** USDC→mUSD swap reverts if output is more than 1% below expected.
- **Reputation re-check at pay.** Payer's score is re-evaluated at the payment moment, not just at invoice creation.
- **Bounded array sizes.** Line items ≤ 50, milestones ≤ 10, mandate counterparties ≤ 50, mandate categories ≤ 20.

## Known limitations

- **Testnet only.** Currently deployed only on Mantle Sepolia. No mainnet deployment.
- **Mock identity registry.** Production EMEI assumes a real ERC-8004 registry. Today, `MockERC8004` allows self-asserted scores.
- **No pause primitive.** Mandates can only be revoked, not paused.
- **No dispute resolution.** `PAID` is terminal. Off-chain dispute mechanisms must be built externally.
- **Owner key risk.** Several contracts have an `owner` who can change `minReputation`, slippage cap, vault config, and weights. Production should multisig or timelock these.

## Reporting issues

For security-sensitive issues, please contact the maintainers privately rather than opening a public issue.

## See also

- [EMEIInvoice authorization](../contracts/emei-invoice.md)
- [Reputation gate](reputation.md)
- [Mandates](mandates.md)
