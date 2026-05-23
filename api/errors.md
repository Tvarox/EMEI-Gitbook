---
description: Every error code returned by the EMEI Facilitator, mapped to its on-chain custom error and a recommended fix.
---

# Errors

All errors return a structured envelope:

```json
{
  "error": {
    "code": "REPUTATION_TOO_LOW",
    "message": "Account 0x... score 200 below threshold 500",
    "details": { "account": "0x...", "score": 200, "threshold": 500 }
  }
}
```

The `code` is stable; build your client switch on it. The `message` is human-readable and may evolve. The `details` shape is per-error.

## Reputation

| Code | HTTP | On-chain error | Cause | Fix |
|---|---|---|---|---|
| `REPUTATION_TOO_LOW` | 409 | `ReputationTooLow(account, score, threshold)` | Either party below `minReputation` at create-time, or payer below at pay-time | Wait for score to recover, or call `setMinReputation` lower (owner only) |

## Invoice

| Code | HTTP | On-chain error | Cause | Fix |
|---|---|---|---|---|
| `INVALID_INVOICE_PARAMS` | 400 | `InvalidInvoiceParams(reason)` | `amount=0`, payer=zero, empty/oversized line items, bad `net_days` | Check the `reason` field |
| `MILESTONE_AMOUNT_MISMATCH` | 400 | `MilestoneAmountMismatch(expected, actual)` | Sum of milestones ≠ invoice amount | Recompute milestones |
| `INVALID_STATUS_TRANSITION` | 409 | `InvalidStatusTransition(current, target)` | Wrong-status call (e.g. `pay` on `ISSUED`) | Check the current status with `GET /emei/invoice/{id}` |
| `INVOICE_NOT_FOUND` | 404 | `InvoiceNotFound(id)` | Bad ID | Verify the ID exists |
| `AMOUNT_MISMATCH` | 409 | `AmountMismatch(expected, actual)` | Settlement amount disagreed | Re-create the invoice |
| `SETTLEMENT_FAILED` | 409 | `SettlementFailed(reason)` | Insufficient allowance/balance, swap slippage exceeded | Check token approval, mint more tokens, or reduce amount |
| `UNAUTHORIZED` | 401 | `Unauthorized()` | Wrong caller for `present` / `pay` / `reject` | Use the right private key |

## Mandate

| Code | HTTP | On-chain error | Cause | Fix |
|---|---|---|---|---|
| `INVALID_MANDATE_PARAMS` | 400 | `InvalidMandateParams(reason)` | Bad time window, oversized arrays, zero cap | Check the `reason` field |
| `MANDATE_NOT_FOUND` | 404 | `MandateNotFound(id)` | Bad ID | Verify the ID exists |
| `MANDATE_NOT_ACTIVE` | 409 | `MandateNotActive(id)` | Mandate is `EXHAUSTED`, `EXPIRED`, or `REVOKED` | Create a new mandate |
| `INSUFFICIENT_MANDATE_CAP` | 409 | `InsufficientMandateCap(remaining, required)` | `remainingCap < amount` | Wait for renewal or split the invoice |
| `MANDATE_EXPIRED` | 409 | `MandateExpired(id)` | `block.timestamp > validUntil` | Create a new mandate |
| `COUNTERPARTY_NOT_APPROVED` | 409 | `CounterpartyNotApproved(addr)` | Issuer not in `approvedCounterparties` | Add issuer to a new mandate, or revoke and re-create |
| `CATEGORY_NOT_APPROVED` | 409 | `CategoryNotApproved(category)` | None of the line-item categories match | Adjust mandate or invoice categories |

## Transport / generic

| Code | HTTP | Cause | Fix |
|---|---|---|---|
| `INVALID_AMOUNT` | 400 | `amount=0` or non-numeric | Send a positive uint256 string |
| `INVALID_ADDRESS` | 400 | Malformed hex | 0x-prefixed, 40 hex chars |
| `MISSING_PRIVATE_KEY` | 401 | `X-Private-Key` header missing on a write endpoint | Add the header |
| `RPC_ERROR` | 502 | Upstream RPC failed | Retry; check `RPC_URL` |
| `INTERNAL_ERROR` | 500 | Unhandled server error | Check Facilitator logs |

## See also

- [Overview & authentication](overview.md)
- [Reputation gate](../concepts/reputation.md)
- [The invoice lifecycle](../concepts/invoice-lifecycle.md)
