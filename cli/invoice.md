---
description: emei invoice subcommands — create, present, pay, get, list.
---

# emei invoice

## `emei invoice create`

```bash
emei invoice create \
  --payer 0xPAYER \
  --amount 100 \
  --asset 0xASSET \
  --terms <due_on_receipt | net_n_days | milestones> \
  [--net-days N] \
  [--milestone "amount:dueDate:description"]... \
  --mode <pay_link | mandate> \
  --line-item "description:amount:category"...
```

| Flag | Required | Notes |
|---|---|---|
| `--payer` | ✅ | Payer's address |
| `--amount` | ✅ | Human-readable; converted using the asset's decimals |
| `--asset` | ✅ | mUSD or USDC contract |
| `--terms` | ✅ | One of `due_on_receipt`, `net_n_days`, `milestones` |
| `--net-days` | only with `net_n_days` | 1–365 |
| `--milestone` | only with `milestones` | Repeatable; sum must equal `--amount` |
| `--mode` | ✅ | `pay_link` or `mandate` |
| `--line-item` | ✅ | Repeatable; format `description:amount:category` |

**Auth:** ✅. The caller becomes the issuer.

## `emei invoice present <id>`

Transition `ISSUED → PRESENTED`. Issuer-only.

## `emei invoice pay <id>`

Pay an invoice in `pay_link` mode. Payer-only. The CLI handles `approve` automatically.

`emei pay <id>` is a shortcut for this.

## `emei invoice get <id>`

Read the full invoice. No auth required.

## `emei invoice list [--from N] [--to N]`

List invoices in an ID range. No auth required.

## See also

- [The invoice lifecycle](../concepts/invoice-lifecycle.md)
- [HTTP API: Invoices](../api/invoices.md)
