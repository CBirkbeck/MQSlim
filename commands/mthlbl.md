---
name: mthlbl
description: Slim shadow of mathlib-quality's /mathlibable. Decide whether a Lean declaration belongs in mathlib. Runs the exhaustive literature + mathlib + composition check, then produces one of five verdict buckets with evidence. Trusts the model to do the search work without per-row required-artifact tables. Use on a single declaration; for files, run sequentially.
---

# /mthlbl — does this belong in mathlib?

Take one Lean declaration and answer: should mathlib have this? The value
is in the search work, not the verdict — and the search is *exhaustive*
on every invocation. Mathlib is Bourbaki 2.0; the right declaration in the
right form takes thorough investigation every time. There is no quick mode.

## Usage

```
/mthlbl Foo.bar                       # one declaration, full workflow
/mthlbl <file.lean>                   # every public decl in file (sequential)
/mthlbl <file1.lean> <file2.lean> ... # multiple files
```

## Prerequisites

- `lake build` clean (the declaration must elaborate).
- ChatGPT MCP available (one of the lit-search channels). Skip with a
  recorded `n/a` if not.
- Optionally `.mathlib-quality/references/` for source-paper notes.

## How to do it (per declaration)

1. **Resolve and read.** Locate the decl in the project; capture its
   signature, docstring, and proof body. Skip and record `SKIP` if any
   of: `private`/`local`, in `test/`/`examples/`, name ends `_aux`,
   docstring starts with `internal`/`auxiliary`, qualified name already
   exists in mathlib (cheap grep of `.lake/packages/mathlib/Mathlib/`).

2. **Write the prose statement.** Convert the Lean type to standard
   mathematical English in 2–4 sentences. This is where the search starts;
   skip this and the search returns nothing.

3. **Exhaustive literature search** (nine channels, all required):
   - **WebSearch ×3** at different generality levels (specific form,
     most-general form, named-after / aliases).
   - **ChatGPT MCP**: ask for the standard form, its generality, and
     historical evolution.
   - `.mathlib-quality/references/` (or record `n/a` with reason).
   - nLab.
   - nCatLab (if categorical).
   - Stacks Project (if algebraic-geometry).
   - MathOverflow / Math.StackExchange.
   - arXiv recent (last 5 years).

   For each channel: record the query, hit/miss, and the standard form
   the channel suggests. A skipped channel without `n/a + reason` is a
   defect.

4. **Generality analysis (per parameter).** For each typeclass / hypothesis
   in the user's form, compare to the literature-standard form. Drop unused
   typeclasses; turn whole-family `∀` into pointwise; weaken
   `[CommRing R]` → `[Ring R]` / `[CommSemiring R]` when the proof allows.
   Verdict: MAXIMALLY GENERAL or STRICTLY NARROWER (with a proposed
   restatement).

5. **Bourbaki 2.0 modern-idiom check.** Even if statement-side is
   "maximally general" relative to literature, ask whether contemporary
   mathlib idioms would re-state with real downstream consequences:
   typeclasses over preambles, filters over sequences, universal
   properties over constructions, bundled types over set-with-closure-
   predicates, modules over vector spaces, higher categories over
   1-categories, generic algebraic-structure indices over concrete ℕ/ℤ/ℝ.

   **Q8 — concrete-via-abstract** (the proof-outruns-the-statement check):
   if the statement mentions a concrete object (`E2`, `π`, a specific
   group) but the proof body, after the first one or two unfolding
   rewrites, never mentions that object again — propose the abstract
   restatement with the concrete result as a one-line corollary.
   Diagnostic: grep the proof body for the named identifier; if it
   appears in 0 or 1 lines after the first `rw`, fire.

6. **Diamond / defeq risk (only for `def`/`class`/`instance`).** Six
   questions: typeclass diamond? reducibility leak? non-canonical
   unfolding? instance priority collision? universe issues? coercion
   ambiguity? Probe with `#synth`, `set_option trace.Meta.synthInstance`,
   `rfl`-defeq probes. Overall risk NONE/LOW/HIGH; HIGH adds a Phase-7
   constraint (rationale must address each HIGH row).

7. **Mathlib search — five methods, on BOTH the user's form AND the
   literature-standard / modern-idiom form:**
   - Lean-Finder (AI search)
   - Loogle (type-pattern)
   - LeanSearch (natural language)
   - grep mathlib source
   - `lean_local_search` (name patterns)

   Conclude: found in mathlib (cite by qualified name) / found more
   general / partial match / building blocks only / not in mathlib.

8. **Composition + call-sites check.**
   - Grep the decl's own call sites in the project (excluding its
     declaring file). Record K = internal use count.
   - Try a ≤3-mathlib-call composition: `.symm`, `.trans` chains, single
     function call, projection chain. Real compositions are mechanical;
     `by rw [..]; ring_nf; aesop` chains are proofs, not compositions.
   - Signals: K ≥ 3 internal uses → real API → leans YES; K = 0 + inline
     re-derivation elsewhere → wrapper consumers bypass → leans NO-
     composable; K = 0 + no re-derivation → BORDERLINE / dead code;
     K = 1 → possibly wrong abstraction.

9. **Verdict** — one of five buckets (see `references/mathlibable-buckets.md`
   for what evidence each needs):
   - `YES-add-as-is`
   - `YES-but-generalise-first` (reason: LITERATURE-WEAKENING or MODERN-IDIOM)
   - `NO-mathlib-has-it`
   - `NO-composable-from-mathlib`
   - `BORDERLINE-needs-human` (with numbered questions)

   **For YES** verdicts: rationale must name the specific mathlib gap /
   TODO / missing-API anchor (not just "searched, didn't find"); propose
   PR location + grouping.
   **For NO** verdicts: rationale must give the concrete refactor plan
   keyed to the K call sites at refactor-actionable detail (the exact
   mathlib decl to call, or the composition sketch ≤3 lines).

   Cost is NOT a verdict factor. EXPENSIVE generalisations are explicitly
   worth doing — mathlib is Bourbaki 2.0.

10. **Persist the full per-decl report** to
    `.mathlib-quality/mathlibable/<decl>.md`. The search is expensive;
    discarding the report is a real loss.

## Mode B: file / multi-file (orchestrator-worker)

When the argument is a file (or several), enumerate every public decl
(handle Lean 4 module syntax: `public def`, `@[expose] public def`,
`@[simp] public lemma`, `noncomputable public def`, etc.), pre-filter the
cheap SKIPs, then dispatch ONE Agent per surviving decl running this full
workflow. Sequential — external API rate limits. Between dispatches,
narrate exactly one scoreboard line.

**Def-first ordering**: assess defs before their dependent lemmas; a glue
lemma (`:= rfl` / `:= Iff.rfl`) inherits the parent's verdict.

**Re-aim**: if a parent def is NO-mathlib-has-it because of more-general
`D'`, dependent lemmas re-aim at `D'` (analogous mathlib lemma exists →
NO; absent → YES-but-generalise-first stated against `D'`) rather than
blanket-inheriting NO.

**Pass `--refs=<absolute path>`** to each worker so they find the
reference docs in the plugin cache.

Aggregate into `MATHLIBABLE_REPORT.md` at the project root.

## What this command does NOT do

- Auto-submit anything to mathlib. The user opens the PR.
- Run the generalisation. YES-but-generalise-first hands off to
  `/genrlz`.
- Decide for the user on BORDERLINE. Numbered questions surface the
  judgement; the user makes the call.
