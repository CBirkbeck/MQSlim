---
name: devel
description: Slim shadow of mathlib-quality's /develop. Plan a mathematical development — search mathlib, design the API, decompose the proof from the source, write a ticket board. Planning-only; workers run via /bstmode. Source-faithful: every leaf must be discharged from a verbatim source quote OR an explicit API gap. Trusts the model to do the literature work without per-step prescriptions.
---

# /devel — plan a mathematical development

Plan a development. Workers don't run here; once the ticket board is
approved, `/bstmode` (or `/beastmode` for the verbose version) picks up the
next available ticket and works it.

## Usage

```
/devel                  # auto-detect: new, resume, or takeover
/devel --continue       # audit existing tickets against the code; propose updates
/devel --status         # show the current ticket board
/devel --decompose      # run only the methodical decomposition pre-work; no tickets yet
```

## Source-faithfulness (binding)

The single most expensive failure this command exists to prevent is a
decomposition that drifts from the source's actual proof. When the plan's
leaves are *invented* (chosen because they "look provable in Lean") rather
than *transcribed* from the reference's own argument, the tree reliably
bottoms out at a lemma the source never proves — and that lemma is then
substantial mathlib-lacking infrastructure or outright false. Every
recurrence has the same root cause: leaving the source.

The discipline:

1. **Transcribe, don't invent.** Read the reference's full proof of each
   result, including every sub-lemma it uses — page-by-page, not
   headline+summary. Your leaves are *its* sub-results, named by their
   mathematical content.
2. **Every leaf carries a precise source locator** — statement number +
   page + line. A leaf with no locator is suspect by default.
3. **Quote-or-delete acceptance test.** For every leaf you must be able to
   quote, verbatim, the source passage that justifies it. If you cannot,
   it is an *artifact*: do not ticket it — go back to the source.
4. **Two red flags that you have left the source — STOP and re-read:**
   the step needs substantial infrastructure absent from mathlib; the
   leaf turns out to be false on inspection.
5. **Never trust memory** for sub-results. Always re-read the source.

## How to do it (planning workflow)

1. **Gather context** — what does the user want proved? What's the source
   (paper, book, blueprint)? What's the scope?

2. **Read the references in full.** The user-provided sources are the
   authoritative target; everything in the plan grounds back to them. If
   the source isn't on disk, ask for it before planning.

3. **Search mathlib exhaustively** for existing definitions and lemmas
   you'd want to reuse. Use Lean-Finder, Loogle, LeanSearch, grep, and
   `lean_local_search`. Note both exact matches and more-general
   versions.

4. **Design the API.** What new definitions? What typeclasses? What
   namespace conventions? Lean toward Bourbaki 2.0 — abstract typeclass
   hierarchies, bundled structures, modern formulations. Mathlib doesn't
   want a vector-space wrapper around module API; don't plan one.

5. **Decomposition pre-work (binding when `--decompose` is set; mandatory
   for any planning that produces tickets).**

   For each top-level result:
   - Read the source's full proof, mirroring its sub-lemma structure.
     Every sub-lemma the source names is a candidate leaf; recurse only
     when the source's proof of a sub-lemma is itself multi-step.
   - Write a prose proof following the source.
   - For each leaf, write the Lean declaration as `:= by sorry`. The
     resulting skeleton must `lake build` clean — if it doesn't, the
     statements are wrong, not the proofs.
   - Quote the verbatim source passage for each leaf + a one-paragraph
     Lean ↔ source match.
   - Verify provability: each leaf either (a) follows from existing
     mathlib (cite by qualified name), (b) follows from project code
     already developed, or (c) is an explicit API gap with its own
     sub-decomposition.

6. **Adversarial attack (mandatory for `--decompose`).** For every leaf,
   try ≥3 distinct attacks across these categories:
   - Counterexample search.
   - Edge-case instantiation.
   - Hypothesis-strength test (would weakening break it?).
   - Source-drift attack (does the leaf actually match the source?).
   - Discharge attack (does the cited mathlib lemma actually apply?).

   For every internal node, attack on composition.

7. **Confidence gate** before writing tickets. Each leaf must have:
   (a) a Lean declaration with `:= by sorry`,
   (b) a verbatim source quote,
   (c) a citation verified against mathlib / project code / API-gap
       sub-decomposition,
   (d) survived ≥3 attacks,
   (e) the source's structure mirrored (not invented), and
   (f) a LOC estimate citing the source's line count.

   Gate fails on any miss. Re-do the leaf.

   **Source-gap fallback.** If the source is opaque on a sub-result,
   cross-reference other references, then search the literature; as a
   last resort, file a numbered question via `/exprvw`. Affected leaves
   enter REVIEW-PENDING and the confidence gate does not pass for them
   until the reviewer's answer is integrated.

8. **Save `decomposition.md`** with the full pre-work artifact.

9. **Write the ticket board** — `.mathlib-quality/tickets.md`. Each ticket
   self-contained: exact Lean statement, numbered proof sketch citing
   sources, the mathlib lemmas needed at each step, the source page +
   line for each step, generality decision.

10. **(Optional) Validate with ChatGPT MCP** — ask for soundness check
    on the plan + the API design.

11. **User approval.** Print the ticket board summary; wait.

## What this command does NOT do

- Execute tickets. That's `/bstmode`.
- Run the cleanup audit. That's `/clnup`.
- Decide whether the result belongs in mathlib. That's `/mthlbl`.

If a sub-result needs external review (the source is opaque, you're not
sure the leaf is provable), surface via `/exprvw` and pause that leaf's
ticket — don't speculate.
