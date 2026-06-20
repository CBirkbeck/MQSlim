---
name: clnup
description: Slim shadow of mathlib-quality's /cleanup. Audit + golf + style + mathlib-replacement on a Lean file (or single declaration). Trusts the model to apply the conventions in skills/mqslim/SKILL.md and to use `lean_diagnostic_messages` / `lean_loogle` / `lean_leansearch` / `lean_local_search` for the audit work. Four hard gates the worker MUST verify before reporting done; everything else is judgement.
---

# /clnup — slim cleanup

Audit + golf a Lean file (or one declaration in it) to mathlib quality.
Style, naming, line packing, mathlib-replacement, generalisation, structure,
inequality orientation. Same intent as `mathlib-quality:cleanup`; trusts you
to do the audit work without per-step prescriptions.

## Usage

```
/clnup MyFile.lean              # whole file
/clnup MyFile.lean foo_thm      # one declaration
```

## Prerequisites

- `lake build` must be clean before you start (otherwise you can't tell
  what you broke vs. what was already broken). Abort with a clear message
  if the baseline is broken.

## How to do it

1. **Read the file in full.** Get a mental model. Note: the conventions in
   `skills/mqslim/SKILL.md` (naming, line packing, `by` placement,
   docstrings, mathlib-first, Bourbaki 2.0) are the rules; reach for them
   without being asked.

2. **File-level fixes first.** Copyright, module docstring (create one if
   missing — list main definitions + main results), imports, strip any
   subsection dividers, drop per-decl `set_option maxHeartbeats`,
   mechanical replacements: `λ` → `fun`, `$` → `<|`, `push_neg` → `push Not`.

3. **Per-declaration golf** — one declaration at a time. For each:
   - Read the proof; understand what it's doing.
   - Apply the golfing patterns you know (inline single-use `have`; squeeze
     non-terminal `simp` only; never squeeze terminal `simp`; prefer `grind`
     / `fun_prop` / `gcongr` / `positivity` / `omega` over manual cases;
     `simpa using` over `simp; exact`; `rwa` over `rw; exact`; etc).
   - Search mathlib for replacements. Use Lean-Finder, Loogle, LeanSearch,
     grep mathlib source, `lean_local_search`. If mathlib has a more
     general version, delete the local lemma and call mathlib directly at
     the call sites.
   - Try mechanical generalisation: drop unused typeclasses; weaken
     `[CommRing R]` to `[Ring R]` / `[CommSemiring R]` when the proof
     allows; pointwise-ify whole-family `∀` hypotheses when the proof only
     uses one instance. Verify with `lean_diagnostic_messages`.
   - If the proof body is >50 lines or has a >10-line branch, flag for
     `/decoproof` rather than golfing in place.

4. **Run the four hard gates** on the diff you've produced (per declaration
   or per file). These catch real failure modes the audit work won't catch
   on its own:
   - `lake_build_file` — must compile.
   - `theorem_statement_protected` — you didn't change the `theorem foo :
     T` line unless an audit item explicitly authorised it (rename per
     `/cleanup`-rename-pass, decomposition split, generalisation).
   - `naming_gate` — every name follows convention (`snake_case` for `Prop`,
     `lowerCamelCase` for data, `UpperCamelCase` for types). Forbidden
     patterns: `\d+_\d+_\d+_`, `m\d+_`, `multipass_`, numeric `_aux\d+`,
     abbreviations (`wt`, `whomog`, `thm`, `eqn`, `imp`, `soln`, `mvpoly`),
     and `_ge_` / `_gt_` (inequality orientation rule). If a rename is
     needed, queue it to `.mathlib-quality/renames.jsonl` for a single
     repo-wide rename pass at the end — never rename in place during golf.
   - `line_packing_gate` — every signature line packed to ~100 chars; no
     line breaks below ~70 chars where the next token would fit (the
     "wide hypothesis" escape applies only when the single hypothesis is
     ≥95 chars on its own line).
   - `inequality_orientation_gate` — Lean code uses `≤` / `<` with smaller
     side on the left, never `≥` / `>`. Rewrite the statement and every
     hypothesis; queue a name rename if the lemma was `_ge_` / `_gt_`.

   See `skills/mqslim/references/cleanup-gates.md` for the gate definitions.

5. **Rename pass** — drain `.mathlib-quality/renames.jsonl` once at the
   end. Dedupe; conflict-check; apply each rename sequentially with
   `Grep` + `Edit replace_all` repo-wide; `lean_diagnostic_messages` after
   each. Truncate the queue when done.

6. **Hand off to the built-in `/simplify` skill** for a holistic pass on
   the changes you made. Re-run the gates if it modified anything.

7. **Report.** One consolidated report:
   - Baseline (`lake build` clean ✓).
   - File-level changes (one short bullet list).
   - Per-declaration changes (one bullet per decl with before/after line
     count + the golfing patterns or mathlib replacements applied).
   - Refactoring done (renames applied, junk inlined, mathlib replacements).
   - Gates: pass/fail status of the five gates above.
   - Flags for follow-up: `/decoproof` candidates, `/genrlz` candidates,
     anything that needs user input.
   - Total line delta.

## Trust + verify

The four gates from step 4 + `lean_diagnostic_messages` clean are the
non-negotiables. Everything else (which simp lemmas to inline, what
counts as a junk wrapper, whether to extract a helper or inline it) is
judgement — exercise it. If you find yourself wanting to write
"PHASE 4 STEP 3.2.b" — stop. State the goal, do the work, report what
you did.

## What this command does NOT do

- Decompose long proofs (>50 lines): flag with `/decoproof <file>
  <decl>` instead.
- Split large files (>1500 lines): flag with `/spltf` instead.
- Big-change generalisations (public-API renames, restatement against a
  different abstract structure, conclusion restatement): flag with
  `/genrlz <file> <decl>` instead — that runs the literature search and
  the user-approval gate.
- Decide whether a declaration belongs in mathlib: that's `/mthlbl`'s
  job.

If you're not sure, flag rather than do — the flag goes in the report.
