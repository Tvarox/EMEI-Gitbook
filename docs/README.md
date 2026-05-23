# Maintainer notes

This directory holds **meta-documentation** about the docs site itself — not user-facing content. It is intentionally **not** linked from `SUMMARY.md`, so GitBook does not render it.

## Files

- [`AUTHORING_PROMPT.md`](AUTHORING_PROMPT.md) — the single, self-contained brief used to generate or regenerate any page on the site. Hand it to an LLM (Claude / GPT / Gemini) along with the page slug you want.

## Workflow

1. **First-time generation:** paste the entire `AUTHORING_PROMPT.md` into an LLM. Replace its §8 block with the page you want, e.g. `"Produce only guides/setup-mandate.md per the spec above."`.
2. **Maintenance:** when the protocol changes, update §2 ("EMEI ground truth") in `AUTHORING_PROMPT.md` and re-run only the affected pages.
3. **Style consistency:** new contributors should skim §4 (per-page template), §5 (GitBook-flavored Markdown), and §6 (quality bar) before writing.

## Why a prompt-driven approach?

Two reasons:

- **LLM-assisted authoring.** A complete brief lets any LLM produce on-style content without re-litigating conventions.
- **Drift control.** When the source of truth (contracts + facilitator) evolves, the docs can be regenerated rather than line-by-line patched.
