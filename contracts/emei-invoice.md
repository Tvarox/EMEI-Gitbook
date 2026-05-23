---
description: EMEIInvoice contract reference — every external function with signature, parameters, returns, reverts, and a runnable cast example.
---

# EMEIInvoice

The central contract. Manages the invoice state machine and orchestrates calls to `Bay8004`, `EMEISettlement`, and `EMEIMandate`.

| | |
|---|---|
| Address (Mantle Sepolia) | [`0xC35f709255D7199394655F16008e8d1A3AD80005`](https://sepolia.mantlescan.xyz/address/0xC35f709255D7199394655F16008e8d1A3AD80005) |
| Source | [`src/EMEIInvoice.sol`](https://github.com/Tvarox/EMEI-Contracts/blob/main/src/EMEIInvoice.sol) |
| Interface | [`src/interfaces/IEMEIInvoice.sol`](https://github.com/Tvarox/EMEI-Contracts/blob/main/src/interfaces/IEMEIInvoice.sol) |
| Solidity | `^0.8.24` |

→ Conceptual model: [The invoice lifecycle](../concepts/invoice-lifecycle.md).

## Types

```solidity
enum Status         { ISSUED, PRESENTED, PAID, OVERDUE, REJECTED }
enum TermType       { DUE_ON_RECEIPT, NET_N_DAYS, MILESTONES }
enum CollectionMode { MANDATE, PAY_LINK }

struct LineItem {
    string  description;
    uint256 amount;
    string  category;
}

struct Milestone {
    uint256 amount;
    uint256 dueDate;
    string  description;
}

struct Terms {
    TermType    termType;
    uint256     netDays;     // [1, 365], used iff termType == NET_N_DAYS
    Milestone[] milestones;  // [1, 10],  used iff termType == MILESTONES
}

struct Invoice {
    uint256        invoiceId;
    address        issuer;
    address        payer;
    uint256        amount;
    address        asset;          // mUSD or USDC
    LineItem[]     lineItems;
    Terms          terms;
    Status         status;
    CollectionMode collectionMode;
    bytes32        settlementProof;
    bytes32        receipt;
    uint256        presentedAt;
    uint256        createdAt;
}

struct CreateInvoiceParams {
    address        payer;
    uint256        amount;
    address        asset;
    LineItem[]     lineItems;
    Terms          terms;
    CollectionMode collectionMode;
}
```

## State

| Variable | Type | Notes |
|---|---|---|
| `owner` | `address` | Admin; can call `setMinReputation` |
| `bay8004` | `address` | The reputation adapter |
| `settlement` | `address` | The settlement contract |
| `mandate` | `address` | The mandate contract |
| `minReputation` | `uint256` | Default `0` (no gate) |

## Functions

### `createInvoice`

```solidity
function createInvoice(CreateInvoiceParams calldata params)
    external
    returns (uint256 invoiceId);
```

Creates a new invoice in `ISSUED` status. Caller becomes `issuer`.

**Parameters:** see `CreateInvoiceParams`.

**Returns:** monotonically incrementing `invoiceId` (starts at `1`).

**Reverts:**
| Error | When |
|---|---|
| `InvalidInvoiceParams("amount must be greater than 0")` | `params.amount == 0` |
| `InvalidInvoiceParams("payer must not be zero address")` | `params.payer == address(0)` |
| `InvalidInvoiceParams("lineItems must not be empty")` | No line items |
| `InvalidInvoiceParams("lineItems exceeds max of 50")` | > 50 line items |
| `InvalidInvoiceParams("netDays must be between 1 and 365")` | `NET_N_DAYS` term with bad `netDays` |
| `InvalidInvoiceParams("milestones must not be empty")` | `MILESTONES` term, empty array |
| `InvalidInvoiceParams("milestones exceeds max of 10")` | > 10 milestones |
| `MilestoneAmountMismatch(expected, actual)` | Sum of milestone amounts ≠ invoice amount |
| `ReputationTooLow(account, score, threshold)` | Either issuer or payer below `minReputation` |

**Events:** `InvoiceCreated`, `StatusChanged`, plus `ReputationChecked` from `Bay8004`.

---

### `present`

```solidity
function present(uint256 invoiceId) external;
```

Transition `ISSUED → PRESENTED`. Records `presentedAt = block.timestamp`.

**Reverts:**
| Error | When |
|---|---|
| `Unauthorized()` | `msg.sender != issuer` |
| `InvalidStatusTransition(current, PRESENTED)` | Invoice not in `ISSUED` |
| `InvoiceNotFound(id)` | Bad ID |

**Events:** `InvoicePresented`, `StatusChanged`.

---

### `pay`

```solidity
function pay(uint256 invoiceId) external;
```

Pay an invoice in pay-link mode. Re-checks payer reputation, calls `EMEISettlement.settle`, transitions to `PAID`.

**Reverts:**
| Error | When |
|---|---|
| `Unauthorized()` | `msg.sender != payer` |
| `ReputationTooLow(payer, score, threshold)` | Payer's score now below threshold |
| `InvalidStatusTransition(current, PAID)` | Invoice not in `PRESENTED` or `OVERDUE` |
| `SettlementFailed(reason)` | Allowance, balance, or slippage failure |

**Events:** `InvoicePaid`, `StatusChanged`, plus `SettlementExecuted` from `EMEISettlement`, plus `ReputationChecked` from `Bay8004`.

---

### `collect`

```solidity
function collect(uint256 invoiceId, uint256 mandateId) external;
```

Permissionless mandate-based collection. Validates the mandate's scope, then performs the same effects as `pay`.

**Caller:** anyone — typically the Auto-Collector.

**Reverts:**
| Error | When |
|---|---|
| `MandateNotActive(mandateId)` | Mandate is `EXHAUSTED`, `EXPIRED`, or `REVOKED` |
| `MandateExpired(mandateId)` | `block.timestamp > validUntil` |
| `CounterpartyNotApproved(addr)` | Issuer not in approved list |
| `CategoryNotApproved(category)` | None of the line-item categories match |
| `InsufficientMandateCap(remaining, required)` | Cap < amount |
| `InvalidStatusTransition(current, PAID)` | Invoice not in `PRESENTED` or `OVERDUE` |
| `ReputationTooLow(payer, score, threshold)` | Payer below threshold |

**Events:** `MandateCollected`, `InvoicePaid`, `StatusChanged`, plus `SettlementExecuted`.

---

### `markOverdue`

```solidity
function markOverdue(uint256 invoiceId) external;
```

Permissionless transition `PRESENTED → OVERDUE`. Only succeeds if the due date has passed.

**Reverts:**
| Error | When |
|---|---|
| `InvalidStatusTransition(current, OVERDUE)` | Not `PRESENTED`, or due date not yet reached |

**Events:** `InvoiceOverdue`, `StatusChanged`.

---

### `reject`

```solidity
function reject(uint256 invoiceId, string calldata reason) external;
```

Issuer-only cancellation. **Terminal** — `REJECTED` is final.

**Reverts:**
| Error | When |
|---|---|
| `Unauthorized()` | `msg.sender != issuer` |
| `InvalidStatusTransition(current, REJECTED)` | Already `PAID` or `REJECTED` |

**Events:** `InvoiceRejected`, `StatusChanged`.

---

### `getInvoice`

```solidity
function getInvoice(uint256 invoiceId) external view returns (Invoice memory);
```

Read the full invoice including line items and milestones.

**Reverts:** `InvoiceNotFound(id)`.

---

### `getInvoiceCount`

```solidity
function getInvoiceCount() external view returns (uint256);
```

Returns the highest assigned `invoiceId`. The next-issued invoice will be `getInvoiceCount() + 1`.

---

### `setMinReputation`

```solidity
function setMinReputation(uint256 threshold) external;
```

Owner-only. Sets the global threshold checked at `createInvoice`, `pay`, and `collect`.

**Reverts:** `Unauthorized()` if `msg.sender != owner`.

## Events

```solidity
event InvoiceCreated(
    uint256 indexed invoiceId,
    address indexed issuer,
    address indexed payer,
    uint256 amount,
    CollectionMode collectionMode
);

event InvoicePresented(
    uint256 indexed invoiceId,
    address indexed payer,
    uint256 presentedAt
);

event InvoicePaid(
    uint256 indexed invoiceId,
    address indexed payer,
    uint256 amount,
    bytes32 settlementProof
);

event InvoiceOverdue(
    uint256 indexed invoiceId,
    address indexed payer
);

event InvoiceRejected(
    uint256 indexed invoiceId,
    string reason
);

event StatusChanged(
    uint256 indexed invoiceId,
    Status previousStatus,
    Status newStatus,
    uint256 timestamp
);
```

## Custom errors

```solidity
error Unauthorized();
error InvalidInvoiceParams(string reason);
error MilestoneAmountMismatch(uint256 expected, uint256 actual);
error InvalidStatusTransition(Status current, Status target);
error InvoiceNotFound(uint256 invoiceId);
error AmountMismatch(uint256 expected, uint256 actual);
error SettlementFailed(string reason);
error ReputationTooLow(address account, uint256 score, uint256 threshold);
```

## Direct on-chain examples (cast)

Read the latest invoice ID:

```bash
cast call --rpc-url https://rpc.sepolia.mantle.xyz \
  0xC35f709255D7199394655F16008e8d1A3AD80005 \
  "getInvoiceCount()(uint256)"
```

Read an invoice's status (returns the enum index):

```bash
cast call --rpc-url https://rpc.sepolia.mantle.xyz \
  0xC35f709255D7199394655F16008e8d1A3AD80005 \
  "getInvoice(uint256)((uint256,address,address,uint256,address,(string,uint256,string)[],(uint8,uint256,(uint256,uint256,string)[]),uint8,uint8,bytes32,bytes32,uint256,uint256))" \
  1
```

Present invoice `1` (issuer key):

```bash
cast send --rpc-url https://rpc.sepolia.mantle.xyz \
  --private-key $ISSUER_KEY \
  0xC35f709255D7199394655F16008e8d1A3AD80005 \
  "present(uint256)" 1
```

Mark invoice `1` overdue (anyone, after due date):

```bash
cast send --rpc-url https://rpc.sepolia.mantle.xyz \
  --private-key $ANY_KEY \
  0xC35f709255D7199394655F16008e8d1A3AD80005 \
  "markOverdue(uint256)" 1
```

## See also

- [The invoice lifecycle](../concepts/invoice-lifecycle.md)
- [HTTP API: Invoices](../api/invoices.md)
- [Events reference](events.md)
- [Errors](../api/errors.md)
