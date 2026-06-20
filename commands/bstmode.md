---
name: bstmode
description: Slim shadow of mathlib-quality's /beastmode. Marathon work session — pick a ticket, finish the goal, stop at nothing. Spawn sub-tickets for missing infrastructure; replan via /devel --continue when the sketch is wrong; mandatory /clnup on every completed proof before mark-done. Trusts the model on the marathon discipline; specifies only the genuine stop conditions and the post-proof cleanup contract.
---

# /bstmode — marathon execution

Pick a ticket from `.mathlib-quality/tickets.md` and finish the goal.
Stop only on the four legitimate stop conditions; everything else is
"keep going". Multi-session work is the *target*, not the exit.

## Usage

```
/bstmode                      # auto-pick next available ticket
/bstmode --ticket T042        # specific ticket
/bstmode --resume             # resume an in_progress ticket from its notes
```

## Identity (binding)

You are not narrating progress. You are not asking for permission. You are
not surfacing meta-observations about how the work is going. You are
proving the ticket. The text you produce is either tool calls or a final
report; mid-turn status reports are forbidden.

## The four legitimate stops

Everything else is "keep going".

1. **DONE.** The original ticket and every sub-ticket it spawned are
   complete. `lean_diagnostic_messages` clean; no `sorry`; `#print axioms`
   shows only the standard set; `lake build` clean; `/clnup` (Phase 6.5
   below) passed.

2. **SCOPE-DEFINITION ERROR.** The statement to be proved is actually
   wrong (counterexample found; the source's setup doesn't survive on
   inspection). Stop and surface to user.

3. **OFF-TRACK.** Drift onto material outside the project's mathematical
   scope, with concrete evidence (the current sub-ticket's mathematics
   contradicts a paragraph in `plan.md`, or the missing infrastructure is
   published-theorem scale). Stop and surface.

4. **BROKEN BASELINE.** `lake build` was broken before you started. Don't
   try to fix project-wide breakage from inside a single ticket.

**None of these stop beastmode**: "this is hard", "it's late", "we're N
sub-tickets deep", "this is taking a while", "this is multi-session work".
Multi-session is the *target* signal — beastmode exists to collapse what
would otherwise be multi-session work into one continuous run.

## How to do it (per ticket)

1. **Pick.** Scan `tickets.md`. Pick the first ticket whose dependencies
   are all `done` and whose status is `open` (or honour `--ticket`).
   Mark `in_progress` with a timestamp.

2. **Arm the Stop hook.** Write `.mathlib-quality/beastmode_active` with
   a one-line FOCUS breadcrumb. The hook (see `hooks/beastmode_stop.sh`
   in the full-fat plugin) keeps the session alive across turn-ends; the
   sentinel's content is overwritten at each meaningful step so a
   re-prompt lands you back where you were.

3. **Read the ticket in full.** Statement, proof sketch, mathlib lemmas
   needed, sources, generality decision, progress notes. If any required
   field is missing, refuse to start — re-run `/devel` to complete the
   ticket plan.

4. **Pre-work checks.**
   - Every `Depends on: TYYY` actually done (compiles, no sorry,
     signature matches). If not done: spawn a sub-ticket for it; recurse.
   - `lake build [target_module]` baseline clean.

5. **State the declaration** verbatim from the ticket's Statement field.
   Don't modify the signature. End with `:= by sorry` so the file
   compiles.

6. **Prove.** Work the proof sketch step by step. For each step:
   - Verify the cited mathlib lemma exists; if not, spawn a sub-ticket
     for the missing infrastructure (Tier A1/A2 spawn).
   - Try the planned tactic. If it fails, diagnose; try variants;
     `lean_multi_attempt` aggressively.
   - If you've made ≥3 honest tactic attempts and the goal won't budge,
     write a sub-ticket for that sub-problem; recurse.
   - Checkpoint progress in the ticket every few meaningful steps.

   **Replan don't stop.** If the sketch turns out to be wrong as a
   strategy — not just one step missing — invoke `/devel --continue` to
   update the ticket plan, then continue from the revised sketch. This is
   not a stop condition; it's a normal part of executing.

7. **Verify.**
   - `lean_diagnostic_messages` clean on every touched file.
   - No `sorry` in the new declarations.
   - `#print axioms <decl>` shows only the standard set (`propext`,
     `Quot.sound`, `Classical.choice`).
   - `lake build` clean.

8. **Gates.** Run the cleanup-gates on the diff: `lake_build_file`,
   `definition_protected`, `theorem_statement_protected`,
   `cumulative_no_unintended_breakage`. Any unauthorised diff =
   unauthorised work; revert.

9. **Phase 6.5 — post-proof `/clnup` (binding).** Before mark-done, invoke

   ```
   Skill(skill="mqslim:clnup", args="<file_path> <decl_name>")
   ```

   for every new declaration this ticket produced. (Use `mathlib-quality:cleanup`
   if you want the verbose version; either way, the contract is the
   same: full cleanup workflow runs to completion before the ticket
   counts as done.)

   The proof is not done until the file is also clean. `/clnup`'s gates
   block ticket completion: a `lake_build_file` ✗ from cleanup means a
   regression you introduced; revert or hard-stop SCOPE-DEFINITION-ERROR.
   A `structure_gate` ✗ with no `/decoproof` flag means the body's too
   long; spawn a `/decoproof` sub-ticket and BLOCK this ticket.
   Rename-queue + simplify-pass-required are non-negotiable.

10. **Mark done.** Update the ticket; append a final progress note;
    re-scan `tickets.md` for the next available ticket.

11. **Sequence-continuation gate.** Immediately after writing the DONE
    update, the next tool call must be `Read .mathlib-quality/tickets.md`
    to find the next ticket. Three outcomes:
    - **Open ticket with met dependencies exists:** continue with that
      ticket. Don't emit a final report yet.
    - **No open tickets with met dependencies; some blocked remain:**
      emit final report with "all dispatchable done; blocked remain".
    - **No open tickets at all:** emit `BEASTMODE-DONE: no open tickets
      with met dependencies.` (verbatim — the loop runtime watches for
      this prefix) followed by the final report.

## Spawning sub-tickets

When you discover missing infrastructure (a mathlib lemma that doesn't
exist; a definition the source assumes but the project doesn't have),
**spawn a sub-ticket in `/devel`'s template format** and recurse:

- Tier A1: missing single mathlib lemma → spawn ticket for it.
- Tier A2: missing definition or larger infrastructure → spawn ticket.
- Tier A3: unmet dependency exists in the board but isn't done yet →
  switch to that ticket; recurse.

Welcome scope growth that stays on-target. A step estimated as two
lemmas turning out to be ten is great news (more mathematics captured),
not a stop signal. The harder the work, the more energy goes in.

## On-target check (before every sub-ticket)

Before spawning a sub-ticket, confirm:
1. It serves the main goal in `plan.md`.
2. It stays in the project's mathematical scope.
3. It's a refinement, not a divergence.

A sub-ticket that drifts onto unrelated mathematics = OFF-TRACK stop,
not a continue.

## Final report shape

```
## /bstmode report — [TXXX] <title>

Status: DONE | <hard-stop reason>
Time: <start> to <end>
Cycles: <approximate retry count>

Sub-tickets spawned: [TYYY, TZZZ, ...] (each done)

Result:
- Declaration: <name> at <file>:<line>
- Lines: <N>
- Mathlib lemmas used: <list>

Verify:
✓ lean_diagnostic_messages clean
✓ no sorry
✓ axioms standard set
✓ lake build clean

Gates: pass (definition_protected, theorem_statement_protected, lake_build_file, cumulative_no_unintended_breakage)

Post-proof cleanup (Phase 6.5):
- Skill(mqslim:clnup) on <file> <decl> — DONE | DONE-with-flags | FAILED-on-<gate>
- Renames applied: <list>
- Decompose flags raised: <list>

Next ticket: <T### or "none ready">
```
