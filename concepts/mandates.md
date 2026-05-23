---
description: How EMEI mandates work — scoped, pre-authorized standing permissions that let invoices be auto-collected without a payer signature per call.
---

# Mandates: scoped auto-collection

A **mandate** is a pre-authorized, scoped standing permission stored on the `EMEIMandate` contract. Once a mandate is active, any invoice that falls within its scope can be settled by calling `EMEIInvoice.collect(invoiceId, mandateId)` — **without a signature from the payer**.

Mandates are EMEI's answer to "I want this agent to bill me for `data-services` up to 1,000 mUSD this month, but I don't want to click *Approve* every time."

## What a mandate is

```solidity
struct Mandate {
    uint256 mandateId;
    address payer;                       // Who's pre-authorizing
    uint256 spendCap;                    // Total ceiling for the period
    uint256 remainingCap;                // Decremented per collection
    address[] approvedCounterparties;    // ≤ 50 addresses
    string[]  approvedCategories;        // ≤ 20 strings
    uint256 validFrom;                   // unix timestamp
    uint256 validUntil;                  // unix timestamp
    MandateStatus status;                // ACTIVE | EXHAUSTED | EXPIRED | REVOKED
}
```

A mandate is **per-payer**. Only the `payer` who created it can revoke it. The validation logic lives entirely on-chain.

## How a collection actually works

When `EMEIInvoice.collect(invoiceId, mandateId)` is called:

1. The invoice is loaded; it must be `PRESENTED` or `OVERDUE`.
2. `EMEIMandate.validateAndDecrement(mandateId, issuer, amount, category)` is called atomically. It checks, in order:
   - Mandate `status == ACTIVE`.
   - `block.timestamp` is within `[validFrom, validUntil]`.
   - `issuer` is in `approvedCounterparties`.
   - At least one of the invoice's line-item categories is in `approvedCategories`.
   - `remainingCap >= amount`.
3. If all checks pass, `remainingCap -= amount` and the function returns `true`.
4. `EMEIInvoice` continues to `EMEISettlement.settle(...)`, then to `Bay8004.giveFeedback(...)`, then transitions the invoice to `PAID`.

Any check failing reverts the **entire** collection. No partial state changes.

```mermaid
sequenceDiagram
    participant Caller as Caller<br/>(typically Auto-Collector)
    participant INV as EMEIInvoice
    participant MAN as EMEIMandate
    participant SET as EMEISettlement
    participant BAY as Bay8004

    Caller->>INV: collect(invoiceId=1, mandateId=7)
    INV->>INV: load invoice; require PRESENTED or OVERDUE
    INV->>MAN: validateAndDecrement(7, issuer, amount, category)
    MAN->>MAN: status, time, counterparty, category, cap checks
    MAN-->>INV: true; remainingCap -= amount
    INV->>SET: settle(invoiceId, payer, payee, amount, asset)
    SET->>SET: transferFrom, swap if USDC, route to vault
    SET-->>INV: settlementProof
    INV->>BAY: giveFeedback(issuer, ...)
    INV->>BAY: giveFeedback(payer, ...)
    INV-->>Caller: (status=PAID emitted)
```

## Status transitions

```
ACTIVE ──cap reaches 0──→ EXHAUSTED
ACTIVE ──validUntil passes──→ EXPIRED
ACTIVE ──revokeMandate(id)──→ REVOKED
```

`EXPIRED` is computed lazily — `getMandate` returns the canonical status by checking `block.timestamp` against `validUntil`. There is no explicit "expire" call.

## Who can call what

| Function | `msg.sender` constraint |
|---|---|
| `createMandate` | Anyone (becomes `payer`) |
| `validateAndDecrement` | **`EMEIInvoice` only.** Reverts otherwise. |
| `revokeMandate` | The mandate's `payer` |
| `getMandate`, `getMandatesByPayer` | Anyone |
| `setInvoiceContract` | Owner |

The "permissionlessness" of `EMEIInvoice.collect` is bounded by the mandate's scope. There is no way for a third party to drain a payer's mandate beyond what the mandate itself allows.

## Limits

| Field | Limit | Why |
|---|---|---|
| `approvedCounterparties` | ≤ 50 addresses | Bounds gas on `validateAndDecrement` |
| `approvedCategories` | ≤ 20 strings | Same |
| `validUntil − validFrom` | No hard cap | But practically, set short. Long mandates increase blast radius if reputation conditions change. |

If you need more than 50 counterparties, create multiple mandates.

## Worked example

> **Scenario:** A finance team wants to allow `agent-A` and `agent-B` to bill them for compute and data, up to 5,000 mUSD over the next 30 days.

```bash
emei mandate create \
  --spend-cap 5000 \
  --counterparties 0xAGENT_A,0xAGENT_B \
  --categories compute,data-services \
  --valid-from $(date +%s) \
  --valid-until $(($(date +%s) + 2592000))
```

Now any invoice that satisfies all of:

- `issuer ∈ {0xAGENT_A, 0xAGENT_B}`
- at least one line-item category in `{compute, data-services}`
- amount ≤ remaining cap
- within the 30-day window

…can be auto-collected by anyone (typically the Facilitator's Auto-Collector running every 10 seconds) without any further interaction from the finance team.

If `agent-C` tries to bill them: `CounterpartyNotApproved`.
If an invoice's only category is `marketing`: `CategoryNotApproved`.
If an invoice asks for 6,000 mUSD: `InsufficientMandateCap(remaining, required)`.

## Common patterns

- **Per-vendor mandates.** One mandate per supplier, each with a tight cap. Easy to revoke a single relationship.
- **Per-category mandates.** Wide counterparty list, narrow category list. Useful when you trust *what* you're buying more than *who* you're buying from.
- **Burst mandates.** Short validity (hours), high cap. For one-off campaigns or batch jobs.
- **Long-tail mandates.** Long validity (months), low cap. For background SaaS-style spend.

## Pause without revoking?

Not directly. `revokeMandate` is terminal. To "pause", revoke and re-create. If you need a pause primitive, build it off-chain in your own logic and revoke the mandate during the pause window.

## Next steps

- [Set up mandate auto-collection](../guides/setup-mandate.md) — the how-to.
- [The invoice lifecycle](invoice-lifecycle.md) — what happens after `validateAndDecrement` returns true.
- [Background services: Auto-Collector](../architecture/background-services.md#auto-collector) — what actually triggers `collect`.

## See also

- [EMEIMandate (contracts reference)](../contracts/emei-mandate.md)
- [HTTP API: Mandates](../api/mandates.md)
- [CLI: emei mandate](../cli/mandate.md)
