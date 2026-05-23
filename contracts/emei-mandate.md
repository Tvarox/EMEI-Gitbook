---
description: EMEIMandate contract reference — scoped pre-authorizations validated only by EMEIInvoice.
---

# EMEIMandate

Stores scoped pre-authorizations and exposes `validateAndDecrement` callable only by `EMEIInvoice`.

| | |
|---|---|
| Address (Mantle Sepolia) | [`0xF48C3bd4FE046629A9c12A39693f39c297893bD8`](https://sepolia.mantlescan.xyz/address/0xF48C3bd4FE046629A9c12A39693f39c297893bD8) |
| Source | [`src/EMEIMandate.sol`](https://github.com/Tvarox/EMEI-Contracts/blob/main/src/EMEIMandate.sol) |
| Interface | [`src/interfaces/IEMEIMandate.sol`](https://github.com/Tvarox/EMEI-Contracts/blob/main/src/interfaces/IEMEIMandate.sol) |

→ Conceptual model: [Mandates](../concepts/mandates.md).

## Types

```solidity
enum MandateStatus { ACTIVE, EXHAUSTED, EXPIRED, REVOKED }

struct Mandate {
    uint256       mandateId;
    address       payer;
    uint256       spendCap;
    uint256       remainingCap;
    address[]     approvedCounterparties;  // ≤ 50
    string[]      approvedCategories;      // ≤ 20
    uint256       validFrom;
    uint256       validUntil;
    MandateStatus status;
}

struct CreateMandateParams {
    uint256   spendCap;
    address[] approvedCounterparties;
    string[]  approvedCategories;
    uint256   validFrom;
    uint256   validUntil;
}
```

## Functions

### `createMandate`

```solidity
function createMandate(CreateMandateParams calldata params)
    external
    returns (uint256 mandateId);
```

Caller becomes the mandate's `payer`.

**Reverts:** `InvalidMandateParams(reason)` for empty/oversized arrays, zero cap, bad time window.

**Events:** `MandateCreated`.

---

### `validateAndDecrement`

```solidity
function validateAndDecrement(
    uint256 mandateId,
    address issuer,
    uint256 amount,
    string calldata category
) external returns (bool);
```

Called **only** by `EMEIInvoice` during `collect`. Atomically validates all scope conditions and decrements `remainingCap`.

**Reverts:**
| Error | When |
|---|---|
| `Unauthorized()` | `msg.sender != invoiceContract` |
| `MandateNotFound(id)` | Bad ID |
| `MandateNotActive(id)` | Status not `ACTIVE` |
| `MandateExpired(id)` | `block.timestamp > validUntil` |
| `CounterpartyNotApproved(issuer)` | Issuer not in approved list |
| `CategoryNotApproved(category)` | Category not in approved list |
| `InsufficientMandateCap(remaining, required)` | `remainingCap < amount` |

**Events:** `MandateCollected` (or `CollectionRejected` on certain failure modes).

---

### `revokeMandate`

```solidity
function revokeMandate(uint256 mandateId) external;
```

Payer-only. Terminal — sets `status = REVOKED`.

**Reverts:** `Unauthorized()`, `MandateNotFound`, `MandateNotActive`.

**Events:** `MandateRevoked`.

---

### Views

```solidity
function getMandate(uint256 mandateId) external view returns (Mandate memory);
function getMandatesByPayer(address payer) external view returns (uint256[] memory);
```

`getMandate` returns the canonical status — it computes `EXPIRED` lazily by comparing `block.timestamp` to `validUntil`.

---

### `setInvoiceContract`

```solidity
function setInvoiceContract(address invoiceContract) external;
```

Owner-only. Configures which address may call `validateAndDecrement`.

## Events

```solidity
event MandateCreated(uint256 indexed mandateId, address indexed payer, uint256 spendCap, uint256 validFrom, uint256 validUntil);
event MandateCollected(uint256 indexed mandateId, uint256 indexed invoiceId, address indexed payer, uint256 amountCollected, uint256 remainingCap);
event CollectionRejected(uint256 indexed mandateId, uint256 indexed invoiceId, string reason);
event MandateRevoked(uint256 indexed mandateId, address indexed payer);
```

## Custom errors

```solidity
error InvalidMandateParams(string reason);
error Unauthorized();
error MandateNotFound(uint256 mandateId);
error MandateNotActive(uint256 mandateId);
error InsufficientMandateCap(uint256 remaining, uint256 required);
error MandateExpired(uint256 mandateId);
error CounterpartyNotApproved(address counterparty);
error CategoryNotApproved(string category);
```

## See also

- [Mandates](../concepts/mandates.md)
- [HTTP API: Mandates](../api/mandates.md)
- [EMEIInvoice.collect](emei-invoice.md#collect)
