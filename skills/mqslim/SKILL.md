---
name: mqslim
description: Slim shadow of the mathlib-quality plugin. Same workflows (cleanup, mathlib-fit assessment, proof decomposition, project development, marathon execution) but trusts frontier-model judgement instead of over-specifying. Use when working with `.lean` files, mathlib contributions, proof golfing, or code review; or when the user types one of the `/clnup` / `/mthlbl` / `/devel` / `/bstmode` / `/decoproof` family of slash commands. Pair with the full mathlib-quality plugin to get both styles loaded; the user picks per task by typing `/cleanup` (verbose, defensively gated) vs `/clnup` (slim, trusts judgement).
trigger:
  filePatterns:
    - "*.lean"
  keywords:
    - mathlib
    - cleanup
    - clnup
    - mathlibable
    - mthlbl
    - decompose
    - decoproof
    - develop
    - devel
    - beastmode
    - bstmode
    - slim
---

# MQSlim — Slim shadow of mathlib-quality

The same workflows the full mathlib-quality plugin codifies, restated in the
"trust the model, don't over-prescribe" style Anthropic's skill-authoring
guide recommends. Frontier models comply over-eagerly with long instruction
lists at the expense of judgement; this plugin keeps the goal + the high-
stakes rails and lets the model reason about the rest.

## When to reach for this vs. the full plugin

- `/clnup` instead of `/cleanup` — when you want a quicker, judgement-led pass
  on a file. Same gates that catch real failure modes (build clean, no statement
  drift, names follow convention, lines packed, inequalities oriented). No
  per-rule status pile-on.
- `/mthlbl` instead of `/mathlibable` — when you want the literature + mathlib
  + composition check + five-bucket verdict without the per-row required-
  artifact tables.
- `/devel`, `/bstmode`, `/decoproof` — analogous shadows of `/develop`,
  `/beastmode`, `/decompose-proof`.

If a task needs maximum enforcement (a big batch run, an unreliable
context, a worker subagent), use the verbose `/cleanup` etc. If you trust
the model with the goal stated cleanly, use the slim one.

## Commands shipped here

| Slim | Shadows |
|------|---------|
| `/clnup` | `/cleanup` |
| `/mthlbl` | `/mathlibable` |
| `/devel` | `/develop` |
| `/bstmode` | `/beastmode` |
| `/decoproof` | `/decompose-proof` |

More to follow (`/overvw`, `/projstat`, `/exprvw`, `/genrlz`, `/presub`,
`/fixfb`, `/bmpml`, `/blprint`, `/unform`, `/contrib`, `/intlrn`, `/spltf`,
`/clnup-all`). Each ports in slim form as the full-fat version proves
stable.

## Core mathlib conventions (the load-bearing rails)

These are the rules workers SHOULD reach for without being asked. Cited
from the mathlib style guide and from accumulated PR-review experience.

**Naming.** `snake_case` for `theorem`/`lemma`/`proposition` (statements
of type `Prop`); `lowerCamelCase` for `def`/`abbrev` (data); `UpperCamelCase`
for types. Hypothesis-ordering convention `C_of_A_of_B` (conclusion +
hypotheses in appearance order). Avoid abbreviations (`wt`, `whomog`, `thm`,
`eqn`, `imp`, `soln`, `mvpoly`).

**Inequality orientation.** Lean code uses `≤` not `≥`, `<` not `>`,
smaller side on the left. Lemma names match: `a_le_b` not `b_ge_a`.

**Line packing.** Fill to ~100 chars; pack multiple parameters per signature
line; don't break at 50-60 when the next token would fit.

**`by` placement.** End of preceding line, never on its own.

**Docstrings.** Public theorems/defs: 1-sentence statement description.
Private/aux: none. No proof narration in docstrings — that goes in `--`
comments inside the proof.

**Mathlib-first.** Before adding a wrapper, search five ways: Lean-Finder,
Loogle, LeanSearch, grep mathlib source, `lean_local_search`. If mathlib
has it, use it directly; don't define a wrapper that just calls the
mathlib version.

**Bourbaki 2.0.** Mathlib is a redo of mathematics with contemporary tools.
The right form is often more abstract than the literature's classical
form (modules not vector spaces; filters not sequences; bundled types
not set-with-closure-predicates). Cost is not a reason to ship the narrow
form when the abstract one is correct.

## References (loaded on demand)

- `references/cleanup-gates.md` — the four hard gates (build, names, line
  packing, inequality orientation) workers should check before finalising
- `references/mathlibable-buckets.md` — the five verdict buckets and what
  evidence each needs (slim version of the verdicts catalogue)

The references are bare-bones; the full mathlib-quality `references/`
folder has the verbose anti-pattern catalogues and worked examples if you
need them.

## Shadow-update contract

When the full-fat mathlib-quality plugin gains a new feature or rule,
MQSlim gets the analogous slim restatement. The bar: capture the
load-bearing intent in 1-3 sentences (or one short bullet list); link to
the full-fat reference for anyone who needs the verbose version.

Version numbers track mathlib-quality's. v0.51.0 here shadows the
mathlib-quality state at the time of MQSlim's creation.
