---
description: End-to-end recipe for an autonomous agent that bills its consumers via EMEI invoices and mandates.
---

# Build an agent that bills automatically

This guide outlines the end-to-end shape of an agent that bills its consumers. The recipe combines mandates (for trusted recurring spend) and pay-links (for one-off consumers).

## Prerequisites

- The Facilitator running.
- Your agent's wallet registered on ERC-8004 with a starting score.
- A clear category taxonomy for what your agent sells (e.g. `compute`, `data-services`, `inference`).

## High-level shape

```
On startup:
  1. Read EMEI_PRIVATE_KEY
  2. Confirm registration via GET /emei/reputation/{me}; if score=0, POST /emei/register

Per consumer interaction:
  3. After the unit of work is delivered:
       - For trusted recurring consumers:
           POST /emei/invoice  with collection_mode=MANDATE
           POST /emei/present  with the invoice_id
           Wait — Auto-Collector handles the rest within ≤10s
       - For new / untrusted consumers:
           POST /emei/invoice  with collection_mode=PAY_LINK
           POST /emei/present
           Return GET /emei/paylink/{id} to the consumer
       - Optimistic: charge after; pessimistic: charge before with a small upfront mandate

Periodically:
  4. GET /emei/balance/{me} to check accrued yield
  5. GET /emei/statement?issuer={me}&event_type=InvoicePaid to reconcile
  6. POST /emei/withdraw when working capital is needed
```

## Key design choices

**Invoice granularity.** One invoice per logical billable event (e.g. one per API call batch, one per inference job). Don't issue an invoice for every single token — gas costs amortize poorly.

**Category strings.** Stable, lowercased, kebab-case (e.g. `data-services`, not `Data Services` or `dataServices`). Mandates match by exact string.

**Idempotency.** Cache `(consumer, work_id) → invoice_id` so retries don't double-bill.

**Reputation hygiene.** Every successful settlement increments your score. Plan for the case where a consumer's score drops mid-flow and `pay` reverts with `ReputationTooLow` — re-queue or `reject`.

## Reference implementations

- The `emei-cli` crate is itself a minimal reference of every endpoint call.
- Polling reconciliation can use the `/emei/statement` endpoint with a saved cursor.

## See also

- [The invoice lifecycle](../concepts/invoice-lifecycle.md)
- [Mandates](../concepts/mandates.md)
- [HTTP API: Overview](../api/overview.md)
