---
description: Pay a mUSD-denominated EMEI invoice using USDC. The Settlement contract auto-swaps with 1% slippage protection.
---

# Pay with USDC (cross-asset)

EMEI lets a payer settle a mUSD-denominated invoice using USDC. `EMEISettlement` performs an on-chain swap with `×1e12` decimal normalization (USDC has 6 decimals, mUSD has 18) and rejects swaps that exceed the 1% slippage cap.

## Prerequisites

- The payer holds enough USDC. → [Get testnet tokens](../getting-started/get-testnet-tokens.md).
- An invoice presented to the payer. The invoice's `asset` field can be either mUSD or USDC; the swap path activates whenever the *payer* sends USDC.

## How it works

```
Payer wallet (USDC)
  → approve(EMEISettlement, amount_usdc)
  → EMEIInvoice.pay(invoiceId)
       → EMEISettlement.settle(...)
            → transferFrom(payer, settlement, amount_usdc)
            → swap to mUSD: amount_usdc × 1e12 → expected_musd
            → require received ≥ expected_musd × 0.99   (1% cap)
            → deposit into payee's vault
```

## Steps

### 1. Approve USDC

```bash
cast send --rpc-url https://rpc.sepolia.mantle.xyz \
  --private-key $PAYER_KEY \
  0x2a2f576b67773d4CFe6157C3150BbAB8fFb0FDF6 \
  "approve(address,uint256)" \
  0xfdCb7bA077069A7Da44711Ee6bdB49174AFA4dD0 \
  100000000   # 100 USDC (6 decimals)
```

### 2. Pay

```bash
emei invoice pay 1
```

The Facilitator detects USDC, signs the approval if missing, and calls `pay`. On success, the issuer's vault holds 100 mUSD (not 100 USDC).

## Slippage failure

If the swap path returns less than `amount_usdc × 1e12 × 0.99`, the call reverts with `SwapFailed(expected, received)` and the invoice stays `PRESENTED`. Common causes:

- The owner has tightened `setSlippageTolerance` below 1%.
- The swap router has been swapped out and the new pool is illiquid.

Retry once or contact the operator.

## See also

- [Settlement & yield](../concepts/settlement.md)
- [EMEISettlement contract](../contracts/emei-settlement.md)
