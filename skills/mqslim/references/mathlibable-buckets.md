# `/mthlbl` verdict buckets — what evidence each needs

Five buckets; each has a non-negotiable evidence requirement. The verdict
is rejected if the evidence isn't there.

## `YES-add-as-is`

The declaration is novel for mathlib at the right generality + non-trivial
+ not composable-from-mathlib in ≤3 calls.

**Evidence required:**
- Lit search across ≥3 channels (the rest recorded `n/a` with reason).
- Generality analysis concluding MAXIMALLY GENERAL (or, if narrower, the
  modern-idiom check Q8 must say "no further abstraction available").
- Diamond/defeq risk: NONE/LOW (HIGH risks explicitly addressed in
  rationale).
- Mathlib search on user's form: no hit.
- Composition check: NOT-COMPOSABLE.
- Call sites: K ≥ 1 (or the rationale explains why a not-yet-consumed
  contribution is still worth shipping — usually because mathlib *should*
  use it elsewhere).
- Rationale **names the specific mathlib gap** — TODO, missing API
  anchor, recurring manual reformulation users do. "Searched, didn't
  find" is not enough.
- PR location + grouping proposed.

## `YES-but-generalise-first`

Novel in some form, but the user's form is strictly narrower than what
should be shipped. Two reasons:

- **LITERATURE-WEAKENING**: Phase 4 found the user's form strictly
  narrower than the literature-standard form.
- **MODERN-IDIOM (Bourbaki 2.0)**: Phase 4c found a contemporary mathlib
  formulation that's a real organisational improvement — composes with
  more of mathlib, eliminates redundancy, enables proofs the old form
  blocked. "Looks cooler in category theory" is not enough; the rationale
  must point at concrete downstream API improvements.

**Evidence required:**
- Lit search identifying the more-general form (LITERATURE-WEAKENING)
  OR Q8 / Q1–Q7 firing with a specific restatement (MODERN-IDIOM).
- Generality analysis with the restatement spelled out as a Lean
  signature.
- Mathlib search on BOTH the user's form AND the generalised form (no
  hit on either).
- Cost: CHEAP / MODERATE / EXPENSIVE. **NOT a verdict downgrade** —
  expensive generalisations are explicitly worth doing.
- Next action: run `/genrlz <decl>` to make the change with the
  literature + modern-idiom targets in mind.

## `NO-mathlib-has-it`

Mathlib already has the result (or a strictly more general one to
specialise from).

**Evidence required:**
- Cite the existing mathlib decl by full qualified name.
- Show the user's form follows in ≤1 line (`example : <stmt> :=
  <mathlib_call> ...`).
- Call sites count K from the project; refactor plan: at each of the K
  sites, replace `<user>.bar` with `<Mathlib.X>.bar` (note any argument-
  order differences — dot notation vs. positional).

## `NO-composable-from-mathlib`

Mathlib has the building blocks but not the exact form; a 1–3-mathlib-call
composition gives the result.

**Evidence required:**
- List the mathlib building blocks by qualified name.
- Composition sketch ≤3 lines.
- The sketch must be a real composition: `.symm`, `.trans`, single
  function call, projection chain. Multi-`have` reasoning with `rw` +
  `ring_nf` + `aesop` is a proof, not a composition — that's a YES.
- Call sites count K; refactor plan: inline the composition at each
  site.

## `BORDERLINE-needs-human`

Phases 1–8 ran cleanly but synthesising them requires a judgement the
worker can't make alone: mathematical taste, project policy, audience-
narrow result, generality vs cost tradeoff, naming conflicts.

**Evidence required:**
- All other phases complete.
- Numbered list of ≤5 concrete questions for the user — each answerable
  yes/no or with a short response.
- Suggested next-step outcomes per likely answer.

A worker citing "uncertain" without spelling out the questions has
skipped the synthesis — re-run Phase 7.

## Verdict gate — what fails it

- Required evidence missing from the bucket above.
- YES-add-as-is but generality analysis said STRICTLY NARROWER or
  Q8 fired (should be YES-but-generalise-first).
- YES-but-generalise-first MODERN-IDIOM with no concrete downstream
  consequences listed ("looks cooler" trap).
- NO-mathlib-has-it without the existing mathlib decl cited by full
  qualified name.
- NO-composable but the sketch is >3 lines or includes real reasoning.
- BORDERLINE without numbered questions.
- One-liner without exemption (defeq-abuse / typeclass-diamond / API-
  stability) shipping YES without the rationale addressing it.
- HIGH-risk `def`/`class`/`instance` shipping YES-add-as-is without
  rationale addressing each HIGH risk.
- Anything other than BORDERLINE citing cost as the reason ("too
  expensive to generalise" is a BORDERLINE question for the user, not
  a self-resolving downgrade).
- Phase 6.0 call-sites table missing or partial.

Failed verdict → re-run Phase 7 only; the Phases 3–6 artifacts are
preserved.
