# MQSlim — Slim shadow of `mathlib-quality`

A parallel Claude Code plugin that does the same work as
[`CBirkbeck/mathlib-quality`](https://github.com/CBirkbeck/mathlib-quality) —
cleanup, mathlib-fit assessment, proof decomposition, project development,
marathon execution — but with skill files restated in the "trust frontier-
model judgement; minimal instructions" style Anthropic's skill-authoring
guide explicitly recommends.

## Why this exists

A repeated observation about the long-form `mathlib-quality` plugin:
frontier models comply over-eagerly with the long instruction lists, at
the expense of mathematical and engineering judgement. The hypothesis here:
restating the same goals in roughly one-fifth the prose lets the model
reason about specifics rather than execute checklists.

## How it pairs with `mathlib-quality`

Load both plugins together. The slim commands are abbreviated so they
don't collide with the full-fat ones:

| Type | Get | Use when |
|------|-----|----------|
| `/cleanup MyFile.lean` | the verbose, gated `mathlib-quality:cleanup` | a big batch run, an unreliable context, or a worker subagent where maximum enforcement helps |
| `/clnup MyFile.lean` | the slim `mqslim:cleanup` | you trust the model with the goal stated cleanly and want less context overhead |

Same for `/mathlibable` vs `/mthlbl`, `/develop` vs `/devel`, `/beastmode`
vs `/bstmode`, `/decompose-proof` vs `/decoproof`.

## Commands shipped (initial set)

| Command | Shadow of |
|---------|-----------|
| `/clnup` | `mathlib-quality:cleanup` |
| `/mthlbl` | `mathlib-quality:mathlibable` |
| `/devel` | `mathlib-quality:develop` |
| `/bstmode` | `mathlib-quality:beastmode` |
| `/decoproof` | `mathlib-quality:decompose-proof` |

More port as the full-fat versions prove stable. See `CLAUDE.md` for the
full list of planned shadows.

## Shadow-update contract

When `mathlib-quality` gains a new feature or refines an existing rule,
MQSlim gets the **analogous slim restatement** — not a literal truncation,
but a re-derivation of the same intent at 1–3 sentences per change. The
discipline (full version in `CLAUDE.md`):

1. Identify the load-bearing intent of the upstream change.
2. State it in the slim file in 1–3 sentences + a one-line example if
   needed.
3. Link out to the full-fat reference for the verbose version.
4. Bump MQSlim's version to parallel `mathlib-quality`'s.

The shadow is not a copy. It's a probe — the goal is to surface what's
*actually load-bearing* in the upstream prompts vs. what's accumulated
over-specification.

## Installation

### Option 1: Claude Code plugin marketplace (recommended)

From within Claude Code:

```
/plugin marketplace add CBirkbeck/MQSlim
/plugin install mqslim
```

The first command tells Claude Code to read the `marketplace.json` at
the repo root and discover the plugins it offers; the second installs
the `mqslim` plugin specifically.

To pair with the full-fat mathlib-quality (recommended — see
"How it pairs" above):

```
/plugin marketplace add CBirkbeck/mathlib-quality
/plugin install mathlib-quality
```

Both load together; the abbreviated `/clnup`-family commands coexist
with the full-name `/cleanup`-family commands. Type whichever you want
per task.

If you also want the Lean 4 theorem-proving workflows the cleanup
commands integrate with:

```
/plugin marketplace add cameronfreer/lean4-skills
/plugin install lean4
```

Restart Claude Code after installation. Verify by typing `/` and
checking that `/clnup`, `/mthlbl`, `/devel`, `/bstmode`, `/decoproof`
appear in the command list.

### Option 2: Local clone

```bash
git clone https://github.com/CBirkbeck/MQSlim.git
```

Then from Claude Code:

```
/plugin marketplace add /absolute/path/to/MQSlim
/plugin install mqslim
```

(The local path must be the directory containing
`.claude-plugin/marketplace.json`.)

### Verifying the install

After installation, in a Lean project session, type `/clnup` and start
typing a filename — autocomplete should suggest the `.lean` files in
your project. If `/clnup` is unknown, the install didn't pick up; try
`/plugin list` to see what's loaded.

### Prerequisites

MQSlim only ships skill files; it has no runtime dependencies. But the
commands themselves rely on:

- **Lean LSP MCP server** — nearly all commands use `lean_diagnostic_messages`,
  `lean_goal`, `lean_loogle`, etc. for sub-second feedback. Set up via
  [`oOo0oOo/lean-lsp-mcp`](https://github.com/oOo0oOo/lean-lsp-mcp);
  full instructions are in
  [mathlib-quality's README](https://github.com/CBirkbeck/mathlib-quality#lean-lsp-mcp-server-highly-recommended).

- **ChatGPT MCP server** (optional, used by `/mthlbl`'s literature
  search) — same setup as for mathlib-quality. See
  [the full-fat README](https://github.com/CBirkbeck/mathlib-quality#optional-chatgpt-mcp-server).

## Status

**Version:** 0.52.0 (parallels mathlib-quality v0.52.0).

## Acknowledgements

The slim style is explicitly modelled on Anthropic's
[Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
guide — "concise is key", progressive disclosure via reference files,
default assumption that Claude is already very smart.

The mathematical content (rules, gates, verdict buckets, methodology) is
shared with — and was developed in — the [`mathlib-quality`](https://github.com/CBirkbeck/mathlib-quality)
plugin. MQSlim adds no new mathematical content; it restates the same
content slim.
