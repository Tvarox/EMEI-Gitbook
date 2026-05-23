---
description: Mint MNT, mUSD, and USDC for testing on Mantle Sepolia.
---

# Get testnet tokens

You'll need three tokens to use EMEI on testnet.

## MNT (gas)

Get MNT from the official Mantle faucet:

> [https://faucet.sepolia.mantle.xyz/](https://faucet.sepolia.mantle.xyz/)

Drip rate and per-day limits are set by the faucet operator.

## mUSD (the protocol's settlement asset)

`MockmUSD` is mintable by anyone (testnet convenience). Up to **1,000,000 mUSD per call**.

{% tabs %}
{% tab title="cast" %}
```bash
cast send \
  --rpc-url https://rpc.sepolia.mantle.xyz \
  --private-key $YOUR_KEY \
  0xb4C74657Ef45AA95E91BBac1db7f9C964D1cAeAD \
  "mint(address,uint256)" \
  $YOUR_ADDRESS \
  1000000000000000000000  # 1000 mUSD (18 decimals)
```
{% endtab %}

{% tab title="ethers.js" %}
```ts
const mUSD = new Contract(
  "0xb4C74657Ef45AA95E91BBac1db7f9C964D1cAeAD",
  ["function mint(address,uint256)"],
  signer
);
await mUSD.mint(yourAddress, parseUnits("1000", 18));
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
mUSD is a **rebasing** token. Once you hold it, your `balanceOf` grows over time as the `rebaseIndex` advances. The mint amount is the *initial* balance, not the final one.
{% endhint %}

## USDC (cross-asset payments)

`MockUSDC` is also publicly mintable, up to **1,000,000 USDC per call** (6 decimals).

```bash
cast send \
  --rpc-url https://rpc.sepolia.mantle.xyz \
  --private-key $YOUR_KEY \
  0x2a2f576b67773d4CFe6157C3150BbAB8fFb0FDF6 \
  "mint(address,uint256)" \
  $YOUR_ADDRESS \
  1000000000  # 1000 USDC (6 decimals)
```

When a payer pays a mUSD-denominated invoice with USDC, the Settlement contract auto-swaps with `×1e12` decimal normalization and a 1% slippage cap. → [Settlement & yield](../concepts/settlement.md)

## Verify your balances

```bash
# mUSD
cast call --rpc-url https://rpc.sepolia.mantle.xyz \
  0xb4C74657Ef45AA95E91BBac1db7f9C964D1cAeAD \
  "balanceOf(address)(uint256)" $YOUR_ADDRESS

# USDC
cast call --rpc-url https://rpc.sepolia.mantle.xyz \
  0x2a2f576b67773d4CFe6157C3150BbAB8fFb0FDF6 \
  "balanceOf(address)(uint256)" $YOUR_ADDRESS
```

## Next steps

- [Quickstart](quickstart.md)
- [Pay with USDC](../guides/pay-with-usdc.md)

## See also

- [Deployed addresses](../contracts/addresses.md)
