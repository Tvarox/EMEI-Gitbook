---
description: Live EMEI contract addresses on Mantle Sepolia.
---

# Deployed addresses

EMEI is currently live on **Mantle Sepolia** (chain ID `5003`) only.

## Core contracts

| Contract | Address | Explorer |
|---|---|---|
| `EMEIInvoice` | `0xC35f709255D7199394655F16008e8d1A3AD80005` | [View](https://sepolia.mantlescan.xyz/address/0xC35f709255D7199394655F16008e8d1A3AD80005) |
| `EMEIMandate` | `0xF48C3bd4FE046629A9c12A39693f39c297893bD8` | [View](https://sepolia.mantlescan.xyz/address/0xF48C3bd4FE046629A9c12A39693f39c297893bD8) |
| `EMEISettlement` | `0xfdCb7bA077069A7Da44711Ee6bdB49174AFA4dD0` | [View](https://sepolia.mantlescan.xyz/address/0xfdCb7bA077069A7Da44711Ee6bdB49174AFA4dD0) |
| `Bay8004` | `0xE61B57D84fb55E2601ab47B83c367612E348d409` | [View](https://sepolia.mantlescan.xyz/address/0xE61B57D84fb55E2601ab47B83c367612E348d409) |
| `EMEIReceipt` | `0x558a20766d5998765B056597b8b78fe1914f3969` | [View](https://sepolia.mantlescan.xyz/address/0x558a20766d5998765B056597b8b78fe1914f3969) |

## Identity & token mocks (testnet)

| Contract | Address | Explorer |
|---|---|---|
| `MockERC8004` | `0x4B560970423B08632bC2Aa31D0a70e29e66Fca37` | [View](https://sepolia.mantlescan.xyz/address/0x4B560970423B08632bC2Aa31D0a70e29e66Fca37) |
| `MockmUSD` | `0xb4C74657Ef45AA95E91BBac1db7f9C964D1cAeAD` | [View](https://sepolia.mantlescan.xyz/address/0xb4C74657Ef45AA95E91BBac1db7f9C964D1cAeAD) |
| `MockUSDC` | `0x2a2f576b67773d4CFe6157C3150BbAB8fFb0FDF6` | [View](https://sepolia.mantlescan.xyz/address/0x2a2f576b67773d4CFe6157C3150BbAB8fFb0FDF6) |

## Network parameters

| Field | Value |
|---|---|
| Chain ID | `5003` |
| RPC URL | `https://rpc.sepolia.mantle.xyz` |
| Block explorer | `https://sepolia.mantlescan.xyz` |
| Faucet (MNT) | `https://faucet.sepolia.mantle.xyz/` |

## Programmatic access

Use these constants in your client code. They are stable for the duration of this testnet deployment.

{% tabs %}
{% tab title="TypeScript" %}
```ts
export const EMEI_ADDRESSES = {
  EMEIInvoice: "0xC35f709255D7199394655F16008e8d1A3AD80005",
  EMEIMandate: "0xF48C3bd4FE046629A9c12A39693f39c297893bD8",
  EMEISettlement: "0xfdCb7bA077069A7Da44711Ee6bdB49174AFA4dD0",
  Bay8004: "0xE61B57D84fb55E2601ab47B83c367612E348d409",
  EMEIReceipt: "0x558a20766d5998765B056597b8b78fe1914f3969",
  MockERC8004: "0x4B560970423B08632bC2Aa31D0a70e29e66Fca37",
  MockmUSD: "0xb4C74657Ef45AA95E91BBac1db7f9C964D1cAeAD",
  MockUSDC: "0x2a2f576b67773d4CFe6157C3150BbAB8fFb0FDF6",
} as const;
```
{% endtab %}

{% tab title="Rust" %}
```rust
use alloy::primitives::address;

pub const EMEI_INVOICE: Address    = address!("C35f709255D7199394655F16008e8d1A3AD80005");
pub const EMEI_MANDATE: Address    = address!("F48C3bd4FE046629A9c12A39693f39c297893bD8");
pub const EMEI_SETTLEMENT: Address = address!("fdCb7bA077069A7Da44711Ee6bdB49174AFA4dD0");
pub const BAY_8004: Address        = address!("E61B57D84fb55E2601ab47B83c367612E348d409");
pub const EMEI_RECEIPT: Address    = address!("558a20766d5998765B056597b8b78fe1914f3969");
pub const MOCK_ERC8004: Address    = address!("4B560970423B08632bC2Aa31D0a70e29e66Fca37");
pub const MOCK_MUSD: Address       = address!("b4C74657Ef45AA95E91BBac1db7f9C964D1cAeAD");
pub const MOCK_USDC: Address       = address!("2a2f576b67773d4CFe6157C3150BbAB8fFb0FDF6");
```
{% endtab %}

{% tab title=".env" %}
```bash
EMEI_INVOICE_ADDRESS=0xC35f709255D7199394655F16008e8d1A3AD80005
EMEI_MANDATE_ADDRESS=0xF48C3bd4FE046629A9c12A39693f39c297893bD8
EMEI_SETTLEMENT_ADDRESS=0xfdCb7bA077069A7Da44711Ee6bdB49174AFA4dD0
BAY_8004_ADDRESS=0xE61B57D84fb55E2601ab47B83c367612E348d409
EMEI_RECEIPT_ADDRESS=0x558a20766d5998765B056597b8b78fe1914f3969
MOCK_ERC8004_ADDRESS=0x4B560970423B08632bC2Aa31D0a70e29e66Fca37
MOCK_MUSD_ADDRESS=0xb4C74657Ef45AA95E91BBac1db7f9C964D1cAeAD
MOCK_USDC_ADDRESS=0x2a2f576b67773d4CFe6157C3150BbAB8fFb0FDF6
```
{% endtab %}
{% endtabs %}

## See also

- [Connect to Mantle Sepolia](../getting-started/connect-mantle.md)
- [Get testnet tokens](../getting-started/get-testnet-tokens.md)
- [Contracts overview](overview.md)
