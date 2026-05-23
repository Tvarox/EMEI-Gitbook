---
description: EMEIReceipt contract reference — Merkle root anchoring for paid-invoice receipt batches.
---

# EMEIReceipt

Stores the Merkle root of each receipt batch posted by the off-chain Receipt Batcher. Exposes `verifyInclusion` for trustless receipt verification by anyone.

| | |
|---|---|
| Address (Mantle Sepolia) | [`0x558a20766d5998765B056597b8b78fe1914f3969`](https://sepolia.mantlescan.xyz/address/0x558a20766d5998765B056597b8b78fe1914f3969) |
| Source | [`src/EMEIReceipt.sol`](https://github.com/Tvarox/EMEI-Contracts/blob/main/src/EMEIReceipt.sol) |
| Interface | [`src/interfaces/IEMEIReceipt.sol`](https://github.com/Tvarox/EMEI-Contracts/blob/main/src/interfaces/IEMEIReceipt.sol) |

→ Conceptual model: [Receipt anchoring](../concepts/receipts.md).

## Functions

### `postMerkleRoot`

```solidity
function postMerkleRoot(uint256 batchNumber, bytes32 merkleRoot) external;
```

**Caller:** the authorized poster (`RECEIPT_BATCHER_PRIVATE_KEY` on the Facilitator).

**Reverts:**
| Error | When |
|---|---|
| `Unauthorized()` | Caller is not the authorized poster |
| `InvalidMerkleRoot()` | `merkleRoot == bytes32(0)` |
| `BatchAlreadyPosted(batchNumber)` | A root has already been posted for this batch |

**Events:** `MerkleRootPosted(batchNumber, merkleRoot, timestamp)`.

---

### `getMerkleRoot`

```solidity
function getMerkleRoot(uint256 batchNumber) external view returns (bytes32);
```

Returns `bytes32(0)` if the batch has not been posted.

---

### `getLatestBatch`

```solidity
function getLatestBatch() external view returns (uint256);
```

Returns the highest posted batch number.

---

### `verifyInclusion`

```solidity
function verifyInclusion(
    uint256 batchNumber,
    bytes32 leaf,
    bytes32[] calldata proof
) external view returns (bool);
```

Verifies that `leaf` (a receipt commitment) is included in the posted Merkle root for `batchNumber`. Pure on-chain verification — independent of the Facilitator.

## Events

```solidity
event MerkleRootPosted(uint256 indexed batchNumber, bytes32 merkleRoot, uint256 timestamp);
```

## Custom errors

```solidity
error Unauthorized();
error InvalidMerkleRoot();
error BatchAlreadyPosted(uint256 batchNumber);
```

## See also

- [Receipt anchoring](../concepts/receipts.md)
- [HTTP API: Receipts](../api/receipts.md)
- [Verify a receipt's Merkle inclusion (guide)](../guides/verify-receipt.md)
