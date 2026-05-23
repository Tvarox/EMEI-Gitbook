---
description: Independently verify an EMEI invoice's receipt was anchored on-chain in a Merkle batch.
---

# Verify a receipt's Merkle inclusion

Anyone can verify that an invoice's receipt was anchored on-chain — without trusting the Facilitator.

## Prerequisites

- A `PAID` invoice ID.
- At least 30 seconds since the payment (so the Receipt Batcher has run at least once after settlement).

## The fast path (Facilitator)

```bash
curl http://localhost:8080/emei/verify/1
```

Response includes `verified: true|false`. The Facilitator computes the leaf, fetches the batch root from `EMEIReceipt`, fetches the Merkle proof from local SQLite, and calls `verifyInclusion` on-chain. Trust required: zero, because the result depends only on the on-chain root.

## The trustless path (direct on-chain)

If you don't trust the Facilitator, reproduce the verification yourself:

1. Compute the leaf locally (`keccak256(abi.encode(invoiceId, payer, amount, settlementProof))` — see source).
2. Pull the batch number and proof from any indexer that stores receipt batches.
3. Fetch the on-chain root:

   ```bash
   cast call --rpc-url https://rpc.sepolia.mantle.xyz \
     0x558a20766d5998765B056597b8b78fe1914f3969 \
     "getMerkleRoot(uint256)(bytes32)" \
     42
   ```

4. Call `verifyInclusion` directly:

   ```bash
   cast call --rpc-url https://rpc.sepolia.mantle.xyz \
     0x558a20766d5998765B056597b8b78fe1914f3969 \
     "verifyInclusion(uint256,bytes32,bytes32[])(bool)" \
     42 \
     0xLEAF \
     "[0x...,0x...]"
   ```

A `true` return is final proof of inclusion in the anchored batch.

## See also

- [Receipt anchoring](../concepts/receipts.md)
- [HTTP API: Receipts](../api/receipts.md)
- [EMEIReceipt contract](../contracts/emei-receipt.md)
