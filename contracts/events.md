---
description: Every event emitted by the five core EMEI contracts plus the mocks. All are indexed for explorer search and the /emei/statement API.
---

# Events

Every state change in EMEI emits an event. The Event Indexer streams all of these into local SQLite for `/emei/statement` queries, and they are also indexed by [Mantlescan](https://sepolia.mantlescan.xyz/).

## EMEIInvoice

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

## EMEIMandate

```solidity
event MandateCreated(
    uint256 indexed mandateId,
    address indexed payer,
    uint256 spendCap,
    uint256 validFrom,
    uint256 validUntil
);

event MandateCollected(
    uint256 indexed mandateId,
    uint256 indexed invoiceId,
    address indexed payer,
    uint256 amountCollected,
    uint256 remainingCap
);

event CollectionRejected(
    uint256 indexed mandateId,
    uint256 indexed invoiceId,
    string reason
);

event MandateRevoked(
    uint256 indexed mandateId,
    address indexed payer
);
```

## EMEISettlement

```solidity
event SettlementExecuted(
    uint256 indexed invoiceId,
    uint256 amount,
    address inputAsset,
    address outputAsset,
    VaultType destinationVault
);

event VaultDepositFailed(
    uint256 indexed invoiceId,
    uint256 amount
);

event VaultPreferenceSet(
    address indexed payee,
    VaultType vault
);

event WithdrawalExecuted(
    address indexed payee,
    uint256 amount,
    VaultType vault
);
```

## Bay8004

```solidity
event ReputationChecked(
    address indexed account,
    uint256 score,
    uint256 threshold
);
```

## EMEIReceipt

```solidity
event MerkleRootPosted(
    uint256 indexed batchNumber,
    bytes32 merkleRoot,
    uint256 timestamp
);
```

## MockERC8004

```solidity
event AgentRegistered(
    address indexed agent,
    uint256 initialScore
);

event FeedbackGiven(
    address indexed subject,
    uint256 indexed invoiceId,
    int256 scoreDelta
);
```

## Querying events

- **Via the Facilitator:** `GET /emei/statement` with filters. → [API: Statements](../api/statements.md).
- **Via Mantlescan:** open each contract's address (linked from the [Deployed addresses](addresses.md) page) and use the **Events** tab.
- **Programmatically:** subscribe with `eth_subscribe` (logs) or poll `eth_getLogs` against the contract addresses.

## See also

- [Event indexer & SQLite schema](../architecture/indexer.md)
- [Query the event statement (guide)](../guides/query-events.md)
