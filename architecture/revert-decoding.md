---
description: How the Facilitator translates contract reverts into structured HTTP error envelopes.
---

# Revert decoding

Solidity custom errors are encoded as 4-byte selectors plus ABI-encoded arguments. The Facilitator maintains a registry of every custom error from the five core contracts and decodes them into the structured error envelope.

## Translation table

| On-chain error | HTTP code | API code | `details` shape |
|---|---|---|---|
| `Unauthorized()` | 401 | `UNAUTHORIZED` | `{}` |
| `InvalidInvoiceParams(string reason)` | 400 | `INVALID_INVOICE_PARAMS` | `{ reason }` |
| `MilestoneAmountMismatch(uint256, uint256)` | 400 | `MILESTONE_AMOUNT_MISMATCH` | `{ expected, actual }` |
| `InvalidStatusTransition(Status, Status)` | 409 | `INVALID_STATUS_TRANSITION` | `{ current, target }` |
| `InvoiceNotFound(uint256)` | 404 | `INVOICE_NOT_FOUND` | `{ invoice_id }` |
| `AmountMismatch(uint256, uint256)` | 409 | `AMOUNT_MISMATCH` | `{ expected, actual }` |
| `SettlementFailed(string)` | 409 | `SETTLEMENT_FAILED` | `{ reason }` |
| `ReputationTooLow(address, uint256, uint256)` | 409 | `REPUTATION_TOO_LOW` | `{ account, score, threshold }` |
| `MandateNotFound(uint256)` | 404 | `MANDATE_NOT_FOUND` | `{ mandate_id }` |
| `MandateNotActive(uint256)` | 409 | `MANDATE_NOT_ACTIVE` | `{ mandate_id }` |
| `MandateExpired(uint256)` | 409 | `MANDATE_EXPIRED` | `{ mandate_id }` |
| `InsufficientMandateCap(uint256, uint256)` | 409 | `INSUFFICIENT_MANDATE_CAP` | `{ remaining, required }` |
| `CounterpartyNotApproved(address)` | 409 | `COUNTERPARTY_NOT_APPROVED` | `{ counterparty }` |
| `CategoryNotApproved(string)` | 409 | `CATEGORY_NOT_APPROVED` | `{ category }` |
| `TransferFailed(address, address, address, uint256)` | 409 | `SETTLEMENT_FAILED` | `{ token, from, to, amount }` |
| `SwapFailed(uint256, uint256)` | 409 | `SETTLEMENT_FAILED` | `{ expected, received }` |
| `InsufficientVaultBalance(address, uint256, uint256)` | 409 | `INSUFFICIENT_BALANCE` | `{ payee, requested, available }` |
| `RegistryUnavailable()` | 502 | `RPC_ERROR` | `{}` |
| `InvalidMerkleRoot()` | 400 | `INVALID_MERKLE_ROOT` | `{}` |
| `BatchAlreadyPosted(uint256)` | 409 | `BATCH_ALREADY_POSTED` | `{ batch_number }` |

Unknown selectors fall through to `INTERNAL_ERROR (500)` with the raw revert data attached for debugging.

## Why this matters

Without decoding, every contract revert would surface as an opaque RPC error. The translation layer turns each revert into a typed, machine-readable response that clients can branch on without parsing free-text messages.

## See also

- [HTTP API: Errors](../api/errors.md)
- Each contract's reference page lists its custom errors.
