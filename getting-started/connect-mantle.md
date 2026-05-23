---
description: Network parameters for Mantle Sepolia, where EMEI is currently deployed.
---

# Connect to Mantle Sepolia

EMEI is currently deployed on **Mantle Sepolia** testnet only. Mainnet deployment is planned but not yet available.

## Network parameters

| Field | Value |
|---|---|
| Network name | Mantle Sepolia |
| Chain ID | `5003` |
| Currency symbol | `MNT` |
| RPC URL | `https://rpc.sepolia.mantle.xyz` |
| Block explorer | `https://sepolia.mantlescan.xyz` |
| Faucet (MNT) | `https://faucet.sepolia.mantle.xyz/` |

## Add to MetaMask

1. Open MetaMask → Networks → **Add a network manually**.
2. Paste the values from the table above.
3. Save.

Or use [Chainlist](https://chainlist.org/?search=mantle+sepolia) for a one-click add.

## Programmatic access

{% tabs %}
{% tab title="ethers.js" %}
```ts
import { JsonRpcProvider } from "ethers";

const provider = new JsonRpcProvider("https://rpc.sepolia.mantle.xyz", {
  chainId: 5003,
  name: "mantle-sepolia",
});
```
{% endtab %}

{% tab title="viem" %}
```ts
import { createPublicClient, http } from "viem";

const client = createPublicClient({
  chain: { id: 5003, name: "Mantle Sepolia", nativeCurrency: { name: "MNT", symbol: "MNT", decimals: 18 } },
  transport: http("https://rpc.sepolia.mantle.xyz"),
});
```
{% endtab %}

{% tab title="alloy-rs" %}
```rust
use alloy::providers::ProviderBuilder;

let provider = ProviderBuilder::new()
    .on_http("https://rpc.sepolia.mantle.xyz".parse()?);
```
{% endtab %}

{% tab title="cast" %}
```bash
cast block-number --rpc-url https://rpc.sepolia.mantle.xyz
```
{% endtab %}
{% endtabs %}

## Next steps

- [Get testnet tokens](get-testnet-tokens.md)
- [Deployed addresses](../contracts/addresses.md)

## See also

- [Mantle official docs](https://docs.mantle.xyz/)
