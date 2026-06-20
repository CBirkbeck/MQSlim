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

As a Claude Code plugin:

```
/plugins
# add the MQSlim entry pointing at this repo
```

Or place the repo under `~/.claude/plugins/` and Claude Code will pick
it up.

## Status

**Version:** 0.51.0 (parallels mathlib-quality v0.51.0 at the time of
MQSlim's creation).

## Acknowledgements

The slim style is explicitly modelled on Anthropic's
[Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
guide — "concise is key", progressive disclosure via reference files,
default assumption that Claude is already very smart.

The mathematical content (rules, gates, verdict buckets, methodology) is
shared with — and was developed in — the [`mathlib-quality`](https://github.com/CBirkbeck/mathlib-quality)
plugin. MQSlim adds no new mathematical content; it restates the same
content slim.
