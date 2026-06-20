---
name: decoproof
description: Slim shadow of mathlib-quality's /decompose-proof. Break long Lean proofs (>50 lines) into focused helper lemmas. Two passes: analysis (identify candidates + write decomposition plans) → decompose (extract helpers, golf them, update the main proof). Five-item helper-hygiene curation per extracted helper before any PR. Trusts the model on the decomposition mathematics; specifies the hygiene rails reviewers catch.
---

# /decoproof — break long proofs into helpers

Mathlib's bar: no proof exceeds 50 lines; main theorems target <15. Long
proofs hide their mathematical structure; decomposing surfaces it and
makes each piece testable and reusable.

## Usage

```
/decoproof <file.lean>             # all proofs >30 lines in the file
/decoproof <file.lean> <theorem>   # just that proof
```

## Prerequisites

- `lake build` clean (you can't tell what you broke without a baseline).

## How to do it

### Pass 1 — Analyse (read carefully, don't rush)

1. **Identify candidates.** Scan the file for proofs >30 lines. Threshold
   reminders: <15 ideal; 15-30 consider; 30-50 should decompose; >50 must
   decompose.

2. **Understand each candidate proof completely** before touching code.
   Read it end-to-end. Answer for each:
   - What is the theorem proving (in plain language)?
   - What are the key mathematical steps?
   - What independent facts are established?
   - What estimates or bounds appear?
   - What `cases` / `by_cases` / `rcases` branches exist? Are they
     independent?
   - Are there repeated patterns across other proofs in this file?

3. **Search mathlib FIRST** before extracting any helper. Many "helpers"
   are already in mathlib. Use Lean-Finder, Loogle, LeanSearch, grep,
   `lean_local_search`. If mathlib has it, use mathlib directly — don't
   extract a wrapper.

4. **Generalise before extracting.** A single-use helper specific to one
   parent theorem is usually the wrong abstraction — try to state it
   weaker / more abstractly so it's useful elsewhere too.

5. **Write a decomposition plan** per candidate: the proposed helpers
   (with names + signatures + 1-line statements), how the main proof will
   read after extraction, what mathlib lemmas they call.

6. **User approval gate.** Print the plans; wait. Decomposition is
   destructive (it rewrites proofs); don't run pass 2 without a green
   light.

### Pass 2 — Decompose (parallel agents OK)

For each approved candidate:

1. Extract each helper as a `private lemma` above the main theorem, with
   the agreed signature. Body: copy the relevant section of the main
   proof.

2. Replace the corresponding section of the main proof with a call to
   the helper.

3. `lean_diagnostic_messages` after each extraction. Fix breakage before
   moving on.

4. Golf the extracted helper. Isolated helpers often become one-liners
   once their context is fixed.

5. Repeat until the main theorem reads as an outline (<15 lines target,
   <30 acceptable).

### Pass 3 — Consolidate

1. Look across newly extracted helpers: any duplicates? Can two similar
   helpers be merged into one parameterised version?

2. Helper visibility audit:
   - Genuinely useful outside this file? → public (no `private`).
   - Used only once and can't generalise? → consider inlining instead.
   - Duplicates something now in mathlib? → use mathlib.

3. Definition audit (after the decomposition, look at the file's defs):
   any now unused? any now obviously matching a mathlib concept? any too
   specific?

4. `lean_diagnostic_messages` on the whole file.

### Pass 3.5 — Helper hygiene (REQUIRED before any PR)

Mechanically-extracted helpers inherit the parent's section context;
reviewers catch every inherited item. Five-item curation per helper:

1. **Generality** — drop unused inherited typeclasses (`[Finite ι]`
   inherited from the section but never used → delete); turn whole-family
   `∀`-hypotheses into pointwise (`(hcop : ∀ i, ¬ p ∣ d i)` when only
   `hcop i₀` is used → `{i : ι} (hcop_i : ¬ p ∣ d i)`).

2. **Real-name encoding** — use mathlib's actual object names
   (`legendreSym` not `legendre`, `qExpansion` not `q`) and encode the
   actual criterion in the helper name, not a vague label
   (`legendreSym_eq_one_of_ncard_primesOver_eq_finrank` not
   `legendre_eq_one_of_complete_split`).

3. **Placement** — if the helper is generic infrastructure (talks about
   `Ideal` / `IsGalois` / abstract typeclasses without mentioning the
   parent's specific concept), move it to the appropriate generic file
   as `public` rather than `private` next to the specialised parent.

4. **Statement-level docstrings** — describe the statement, not the
   proof. Proof narration goes in `--` comments inside the proof body.

5. **Document brittle inherited steps** — if the helper's proof has a
   step depending on a definitional equality, a `fun _ => rfl` defeq
   across a wrapper, a `show … by ring` reshape — add a one-line
   `-- BRITTLE: …` comment explaining what's fragile.

Per-helper curation report (one row per extracted helper) is the
required pass-3.5 artifact.

## Helper design rules (the rails)

- `snake_case` names describing what the helper proves.
- `_aux` suffix for helpers specific to one parent theorem; `private`.
- Helpers genuinely useful elsewhere: public, no `_aux`.
- Minimal hypotheses (weaker = more reusable).
- Golf aggressively after extraction.
- Mathematical names ("`norm_bound_of_continuous_on_compact`"), not
  structural ones ("`main_theorem_aux1`", "`step_2`").

## What this command does NOT do

- Style + golf the helpers post-extraction at full depth: that's
  `/clnup`. After `/decoproof` finishes, run `/clnup <file>` to take the
  helpers through the audit + gates.
- Decide whether the helpers belong in mathlib: that's `/mthlbl` on
  each helper.
- File splitting (when the file >1500 lines after decomposition): that's
  `/spltf`.

If a candidate proof is genuinely irreducible case-work after honest
extraction attempts, flag it with a one-line reason in the report — not
every long proof decomposes.
