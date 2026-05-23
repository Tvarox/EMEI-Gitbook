---
description: Common errors when integrating with EMEI and how to resolve them.
---

# Troubleshooting

## My invoice creation reverts with `ReputationTooLow`

Either the issuer or the payer is below the current `minReputation` threshold on `EMEIInvoice`. Check both:

```bash
emei reputation 0xISSUER
emei reputation 0xPAYER
```

If either is unregistered (`score: 0`), call `emei wallet create` for that key. If they're registered but below threshold, give them feedback by completing low-stakes invoices first, or have the contract owner lower the threshold.

→ [Reputation gate](../concepts/reputation.md).

## The Auto-Collector isn't picking up my mandate-mode invoice

Walk down the validation chain in order:

1. Is the invoice actually `PRESENTED` or `OVERDUE`? `emei invoice get <id>`.
2. Is the mandate `ACTIVE`? Check `getMandate` on-chain or the `/emei/statement?mandate_id=N` audit trail.
3. Does the mandate's `validUntil` cover `block.timestamp`?
4. Is the invoice's issuer in the mandate's `approvedCounterparties`?
5. Is at least one of the invoice's line-item categories in the mandate's `approvedCategories`? Check exact case and spelling.
6. Is `remainingCap >= invoice.amount`?

Also check `/emei/statement?event_type=CollectionRejected` for the structured reason.

Finally, verify the operator wallet has MNT for gas:

```bash
cast balance --rpc-url https://rpc.sepolia.mantle.xyz $OPERATOR_ADDRESS
```

## `emei invoice pay` reverts with `SwapFailed`

The USDC→mUSD swap exceeded the slippage cap. Possible causes:

- The owner has tightened `setSlippageTolerance` below 1%.
- The configured swap router is illiquid for your amount.

Try a smaller invoice or contact the operator.

## `emei invoice pay` reverts with `TransferFailed`

The payer hasn't approved enough of the invoice asset to `EMEISettlement`. The CLI handles approvals automatically in most cases, but if you're calling the contract directly:

```bash
cast send --rpc-url https://rpc.sepolia.mantle.xyz \
  --private-key $PAYER_KEY \
  $ASSET_ADDRESS \
  "approve(address,uint256)" \
  0xfdCb7bA077069A7Da44711Ee6bdB49174AFA4dD0 \
  $AMOUNT
```

## My `/emei/statement` query returns empty results

Two possibilities:

1. **The Indexer hasn't caught up.** Compare the latest indexed block to RPC head:
   ```bash
   sqlite3 ./emei.db "SELECT MAX(last_block) FROM indexer_checkpoint;"
   cast block-number --rpc-url https://rpc.sepolia.mantle.xyz
   ```
2. **Your filters are too narrow.** Try the same query without filters and confirm events exist.

## Contract calls succeed but `/emei/invoice/{id}` returns 404

The Indexer hasn't picked up the `InvoiceCreated` event yet. Wait one block and retry. If the gap persists, check the Facilitator logs for indexer errors.

## My Receipt Batcher transactions revert with `Unauthorized`

The `RECEIPT_BATCHER_PRIVATE_KEY` address has not been authorized on `EMEIReceipt`. The contract owner must call the authorization function before the batcher can post roots.

## `verifyInclusion` returns `false` for a paid invoice

Two possibilities:

1. **Receipt not yet batched.** Wait at least 30 seconds after the `PAID` event and retry.
2. **Wrong batch number.** `/emei/verify/{id}` looks up the batch in SQLite. If the local DB lost the assignment, query `/emei/statement?event_type=MerkleRootPosted` and trace by timestamp.

## See also

- [HTTP API: Errors](../api/errors.md)
- [Background services](../architecture/background-services.md)
- [FAQ](faq.md)
