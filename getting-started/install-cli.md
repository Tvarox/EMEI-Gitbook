---
description: Build and configure the emei CLI from source. Single binary, talks to the Facilitator over HTTP.
---

# Install the CLI

The `emei` CLI is a thin HTTP client built from the `emei-cli` crate. It emits structured JSON suited for piping into agent runtimes.

## Prerequisites

- **Rust 1.75+** (`rustup install stable`)
- A clone of [`EMEI-Facilitator`](https://github.com/Tvarox/EMEI-Facilitator)

## Build

```bash
git clone https://github.com/Tvarox/EMEI-Facilitator.git
cd EMEI-Facilitator
cargo build -p emei-cli --release
```

The binary lands at `target/release/emei`. Add it to your `PATH`:

```bash
export PATH="$PWD/target/release:$PATH"
```

Confirm:

```bash
emei --help
```

## Configure

The CLI reads two environment variables:

| Variable | Required | Default | Description |
|---|---|---|---|
| `EMEI_API_URL` | yes | — | Base URL of the Facilitator, e.g. `http://localhost:8080`. |
| `EMEI_PRIVATE_KEY` | yes (for ✅ commands) | — | Hex-encoded private key, with or without `0x` prefix. Sent as the `X-Private-Key` header. |

Example:

```bash
export EMEI_API_URL=http://localhost:8080
export EMEI_PRIVATE_KEY=0xYOUR_DISPOSABLE_TESTNET_KEY
```

{% hint style="warning" %}
Use **testnet-only** keys. The CLI sends the private key to the Facilitator. The Facilitator is intended to run on infrastructure you control; it is not a custodial service.
{% endhint %}

## Verify the connection

```bash
emei reputation 0xb4C74657Ef45AA95E91BBac1db7f9C964D1cAeAD
```

Returns the score for the queried address (or `0` if unregistered).

## Next steps

- [Run the Facilitator locally](run-facilitator.md)
- [Quickstart](quickstart.md)

## See also

- [CLI reference](../cli/overview.md)
