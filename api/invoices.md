---
description: Full HTTP reference for invoice operations — create, present, pay, collect, and read.
---

# Invoices

Five endpoints cover the entire invoice lifecycle: `POST /emei/invoice` (create), `POST /emei/present`, `POST /emei/pay`, `POST /emei/collect`, and `GET /emei/invoice/{id}`.

→ Conceptual model: [The invoice lifecycle](../concepts/invoice-lifecycle.md).
→ State machine and authorization rules are enforced on-chain by [`EMEIInvoice`](../contracts/emei-invoice.md).

---

## POST /emei/invoice

Create a new invoice. Reverts on the chain if either issuer or payer is below `minReputation`.

**Auth:** ✅ `X-Private-Key` (becomes the issuer)

### Request

```json
{
  "payer": "0xPAYER_ADDRESS",
  "amount": "100000000000000000000",
  "asset": "0xb4C74657Ef45AA95E91BBac1db7f9C964D1cAeAD",
  "line_items": [
    { "description": "API access (1000 calls)", "amount": "100000000000000000000", "category": "data-services" }
  ],
  "terms": {
    "term_type": "NET_N_DAYS",
    "net_days": 7,
    "milestones": []
  },
  "collection_mode": "PAY_LINK"
}
```

| Field | Type | Required | Constraints |
|---|---|---|---|
| `payer` | address | ✅ | Non-zero |
| `amount` | uint256 string | ✅ | > 0 |
| `asset` | address | ✅ | mUSD or USDC contract address |
| `line_items` | array | ✅ | 1–50 items. Sum of `amount`s SHOULD equal invoice `amount`. |
| `line_items[].description` | string | ✅ | Free-form |
| `line_items[].amount` | uint256 string | ✅ | > 0 |
| `line_items[].category` | string | ✅ | Used for mandate matching |
| `terms.term_type` | enum | ✅ | `DUE_ON_RECEIPT` \| `NET_N_DAYS` \| `MILESTONES` |
| `terms.net_days` | uint256 | only if `NET_N_DAYS` | 1–365 |
| `terms.milestones` | array | only if `MILESTONES` | 1–10 items; sum of amounts MUST equal invoice `amount` |
| `collection_mode` | enum | ✅ | `MANDATE` \| `PAY_LINK` |

### Response (201 Created)

```json
{
  "invoice_id": 1,
  "tx_hash": "0xabc...",
  "block_number": 12345678,
  "status": "ISSUED",
  "issuer": "0xISSUER",
  "payer": "0xPAYER",
  "amount": "100000000000000000000",
  "created_at": 1716000000
}
```

### Common errors

| HTTP | Code | When |
|---|---|---|
| 400 | `INVALID_INVOICE_PARAMS` | `amount = 0`, payer is zero, line items empty, > 50 line items, `net_days` out of range, milestones don't sum, > 10 milestones |
| 400 | `MILESTONE_AMOUNT_MISMATCH` | Sum of milestone amounts ≠ invoice `amount` |
| 409 | `REPUTATION_TOO_LOW` | Either party below `minReputation` |

### Examples

{% tabs %}
{% tab title="curl" %}
```bash
curl -X POST http://localhost:8080/emei/invoice \
  -H "X-Private-Key: $ISSUER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "payer": "0xPAYER",
    "amount": "100000000000000000000",
    "asset": "0xb4C74657Ef45AA95E91BBac1db7f9C964D1cAeAD",
    "line_items": [{"description":"API","amount":"100000000000000000000","category":"data-services"}],
    "terms": {"term_type":"NET_N_DAYS","net_days":7,"milestones":[]},
    "collection_mode": "PAY_LINK"
  }'
```
{% endtab %}

{% tab title="emei CLI" %}
```bash
emei invoice create \
  --payer 0xPAYER \
  --amount 100 \
  --asset 0xb4C74657Ef45AA95E91BBac1db7f9C964D1cAeAD \
  --terms net_n_days --net-days 7 \
  --mode pay_link \
  --line-item "API access:100:data-services"
```
{% endtab %}

{% tab title="TypeScript" %}
```ts
const res = await fetch("http://localhost:8080/emei/invoice", {
  method: "POST",
  headers: {
    "X-Private-Key": process.env.ISSUER_KEY!,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    payer: "0xPAYER",
    amount: "100000000000000000000",
    asset: "0xb4C74657Ef45AA95E91BBac1db7f9C964D1cAeAD",
    line_items: [{ description: "API", amount: "100000000000000000000", category: "data-services" }],
    terms: { term_type: "NET_N_DAYS", net_days: 7, milestones: [] },
    collection_mode: "PAY_LINK",
  }),
});
const { invoice_id } = await res.json();
```
{% endtab %}
{% endtabs %}

---

## POST /emei/present

Transition `ISSUED → PRESENTED`. Records `presentedAt` and starts the due-date clock.

**Auth:** ✅ `X-Private-Key` — must equal the invoice's `issuer`.

### Request

```json
{ "invoice_id": 1 }
```

### Response (200 OK)

```json
{
  "invoice_id": 1,
  "tx_hash": "0x...",
  "status": "PRESENTED",
  "presented_at": 1716000123
}
```

### Common errors

| HTTP | Code | When |
|---|---|---|
| 401 | `UNAUTHORIZED` | Caller is not the issuer |
| 409 | `INVALID_STATUS_TRANSITION` | Invoice not in `ISSUED` |
| 404 | `INVOICE_NOT_FOUND` | No invoice with that ID |

---

## POST /emei/pay

Pay an invoice in pay-link mode. Re-checks payer reputation, calls `EMEISettlement.settle`, transitions to `PAID`.

**Auth:** ✅ `X-Private-Key` — must equal the invoice's `payer`.

The Facilitator handles the ERC-20 approval automatically (`approve(settlement, amount)` on the invoice's `asset`) before calling `pay`.

### Request

```json
{ "invoice_id": 1 }
```

### Response (200 OK)

```json
{
  "invoice_id": 1,
  "tx_hash": "0x...",
  "status": "PAID",
  "amount": "100000000000000000000",
  "settlement_proof": "0xabc...",
  "paid_at": 1716000456
}
```

### Common errors

| HTTP | Code | When |
|---|---|---|
| 401 | `UNAUTHORIZED` | Caller is not the payer |
| 409 | `REPUTATION_TOO_LOW` | Payer's score dropped below `minReputation` since creation |
| 409 | `INVALID_STATUS_TRANSITION` | Invoice already `PAID`, `REJECTED`, or still `ISSUED` |
| 409 | `SETTLEMENT_FAILED` | Insufficient balance/allowance, slippage exceeded for USDC |

---

## POST /emei/collect

Trigger mandate auto-collection. **Permissionless** — anyone can call. Validation lives in `EMEIMandate.validateAndDecrement`.

**Auth:** ❌ No private key required. The Facilitator signs with its `OPERATOR_PRIVATE_KEY`.

### Request

```json
{
  "invoice_id": 1,
  "mandate_id": 7
}
```

### Response (200 OK)

Same shape as `POST /emei/pay`, plus `mandate_id` and `remaining_cap`.

```json
{
  "invoice_id": 1,
  "mandate_id": 7,
  "tx_hash": "0x...",
  "status": "PAID",
  "amount": "100000000000000000000",
  "settlement_proof": "0xabc...",
  "remaining_cap": "4900000000000000000000"
}
```

### Common errors

| HTTP | Code | When |
|---|---|---|
| 409 | `MANDATE_NOT_ACTIVE` | Status is `EXHAUSTED`, `EXPIRED`, or `REVOKED` |
| 409 | `MANDATE_EXPIRED` | `block.timestamp` past `validUntil` |
| 409 | `COUNTERPARTY_NOT_APPROVED` | Issuer not in `approvedCounterparties` |
| 409 | `CATEGORY_NOT_APPROVED` | None of the invoice's line-item categories match |
| 409 | `INSUFFICIENT_MANDATE_CAP` | `remainingCap < amount` |
| 409 | `INVALID_STATUS_TRANSITION` | Invoice not in `PRESENTED` or `OVERDUE` |

---

## GET /emei/invoice/{id}

Fetch a single invoice's full state.

**Auth:** ❌

### Path parameters

| Name | Type |
|---|---|
| `id` | `uint256` |

### Response (200 OK)

```json
{
  "invoice_id": 1,
  "issuer": "0xISSUER",
  "payer": "0xPAYER",
  "amount": "100000000000000000000",
  "asset": "0xb4C74657Ef45AA95E91BBac1db7f9C964D1cAeAD",
  "status": "PAID",
  "collection_mode": "PAY_LINK",
  "settlement_proof": "0xabc...",
  "receipt": "0xdef...",
  "line_items": [
    { "description": "API access", "amount": "100000000000000000000", "category": "data-services" }
  ],
  "terms": {
    "term_type": "NET_N_DAYS",
    "net_days": 7,
    "milestones": []
  },
  "created_at": 1716000000,
  "presented_at": 1716000123
}
```

### Common errors

| HTTP | Code | When |
|---|---|---|
| 404 | `INVOICE_NOT_FOUND` | No invoice with that ID |

---

## See also

- [The invoice lifecycle](../concepts/invoice-lifecycle.md) — what each transition does on-chain.
- [EMEIInvoice (contracts reference)](../contracts/emei-invoice.md) — the underlying contract API.
- [Errors](errors.md) — every error code, mapped to its on-chain custom error.
- [CLI: emei invoice](../cli/invoice.md) — equivalent CLI commands.
