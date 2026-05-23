---
description: Reference for the three testnet mocks — MockERC8004, MockmUSD, MockUSDC.
---

# Mocks (mUSD, USDC, ERC8004)

Three mock contracts exist on Mantle Sepolia to replace production primitives that aren't deployable on testnet (ERC-8004 registry, yield-bearing stablecoin) or for cost-free testing (USDC).

## MockERC8004 — Identity & reputation registry

| | |
|---|---|
| Address | [`0x4B560970423B08632bC2Aa31D0a70e29e66Fca37`](https://sepolia.mantlescan.xyz/address/0x4B560970423B08632bC2Aa31D0a70e29e66Fca37) |
| Interface | [`IMockERC8004.sol`](https://github.com/Tvarox/EMEI-Contracts/blob/main/src/interfaces/IMockERC8004.sol) |

```solidity
function register() external;
function register(uint256 initialScore) external;       // score capped at 10000
function scoreOf(address) external view returns (uint256);
function giveFeedback(address subject, uint256 invoiceId, uint256 amount) external;
function isRegistered(address) external view returns (bool);
```

**Events:** `AgentRegistered`, `FeedbackGiven`.
**Errors:** `AlreadyRegistered`, `NotRegistered`.

The `register(initialScore)` overload is a testnet convenience. On a production ERC-8004 deployment, scores are earned via `giveFeedback`, not self-asserted.

## MockmUSD — Yield-bearing stablecoin (18 decimals)

| | |
|---|---|
| Address | [`0xb4C74657Ef45AA95E91BBac1db7f9C964D1cAeAD`](https://sepolia.mantlescan.xyz/address/0xb4C74657Ef45AA95E91BBac1db7f9C964D1cAeAD) |

Shares-based rebasing ERC-20. `balanceOf = shares × rebaseIndex / 1e18`.

```solidity
function mint(address to, uint256 amount) external;     // up to 1,000,000 per call
function setRebaseAPY(uint256 bps) external;            // owner-only
function rebase() external;
function sharesOf(address) external view returns (uint256);
```

`rebaseIndex` advances based on `apyBps` and time. Calling `rebase()` makes the advance explicit; transfers also trigger lazy rebase.

## MockUSDC — Standard stablecoin (6 decimals)

| | |
|---|---|
| Address | [`0x2a2f576b67773d4CFe6157C3150BbAB8fFb0FDF6`](https://sepolia.mantlescan.xyz/address/0x2a2f576b67773d4CFe6157C3150BbAB8fFb0FDF6) |

Standard OpenZeppelin ERC-20 with public mint:

```solidity
function mint(address to, uint256 amount) external;     // up to 1,000,000 per call
```

## See also

- [Get testnet tokens](../getting-started/get-testnet-tokens.md)
- [Settlement & yield](../concepts/settlement.md)
- [Reputation gate](../concepts/reputation.md)
