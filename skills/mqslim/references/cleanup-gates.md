# `/clnup` gates — what catches real failure modes

Five gates the worker MUST verify on the diff before reporting done.
The first two catch *unwanted* changes (you accidentally drifted the
signature); the last three catch *missing* work (you didn't actually
apply the rule).

## `lake_build_file`

The file (and the modules that import it) must compile. Run
`lake build <module>`; exit code 0 is the pass.

## `theorem_statement_protected`

The `theorem <decl> : <type>` line should be unchanged unless an audit
item explicitly authorised the change. Authorised reasons:
- a rename (item 5 / Phase-5b rename pass — the name changed, the
  type didn't),
- a structure split (item 12 — `∧` extracted into separate lemmas; the
  type changes but in a documented way),
- a generalisation (item 18 — small typeclass weakening that survives
  all other gates).

A `theorem` line change that traces to none of those = unwanted drift =
gate fail = revert.

## `naming_gate`

Every declaration name follows convention.

- `snake_case` for things of type `Prop` (`theorem`/`lemma`/`proposition`)
- `lowerCamelCase` for `def`/`abbrev` (returning data)
- `UpperCamelCase` for `structure`/`inductive`/`class`

Forbidden patterns:
- `\d+_\d+_\d+_` (e.g. `miyake_4_6_5_…`) — scheme-number names
- `m\d+_` (e.g. `m6_2_…`) — internal-section names
- `multipass_` — internal-section names
- numeric `_aux\d+` (e.g. `_aux1`, `_aux2`) — bare `_aux` is the
  encouraged suffix for `private` helpers; only the numbered variant is
  forbidden
- abbreviations: `wt`, `whomog`, `thm`, `eqn`, `imp`, `soln`, `mvpoly`
- `_ge_`, `_gt_` — orientation rule, see `inequality_orientation_gate`

**Renames defer to the Phase-5b rename pass.** Phase-4 workers append
`{old, new, scope, file, …}` to `.mathlib-quality/renames.jsonl` instead
of renaming in place. The rename pass runs once at the end, drains the
queue, applies each rename sequentially repo-wide. Parallel Phase-4
workers renaming in place collide on shared call-site files — the queue
serialises.

PASSES if (a) no forbidden pattern OR (b) a rename is queued for every
offending name.

## `line_packing_gate`

Every signature line packed to ~100 chars; line breaks below ~70 chars
where the next token would fit are rejected.

The exception: a line whose single hypothesis is ≥95 chars on its own
can legitimately stand below 100 (no neighbour fits without overflow).
Document the exception per-line: `<line N>: 87 chars; single hyp 95
chars; next token would push to 102+`.

The arithmetic is `current_chars + 1 + next-token-width ≤ 100`. If
true, the line is underpacked → repack. If false, the line is correctly
packed. Generic "ok" without the arithmetic = gate fail.

## `inequality_orientation_gate`

In Lean code (signature + every hypothesis), every `≥` must be rewritten
as `≤` with sides swapped; every `>` as `<` with sides swapped. Lemma
names containing `_ge_` or `_gt_` queue a rename (`a_ge_b` → `b_le_a`).

Docstrings and `--` comments are out of scope. The rule is about Lean
code, where uniformity of orientation matters for `simp` lemma matching.

PASSES iff post-edit signature has zero `≥` / `>` in Lean code AND any
`_ge_` / `_gt_` name has a rename queued.

## Recovery (when a gate fails)

- **`lake_build_file` ✗ or `theorem_statement_protected` ✗** (unwanted
  drift): revert the offending edit; retry without it.
- **`naming_gate` / `line_packing_gate` / `inequality_orientation_gate`
  ✗** (missing work): do the missing work. Don't revert — these are
  about producing the right shape, not about preserving the wrong one.

In every case, never ship a `Gates: FAIL`. If you genuinely can't
resolve a gate this run, the resolution IS the explicit flag in the
report ("Refactoring needed: rename <old> → <new> queued for Phase 5b"
or "structural decomposition needed: `/decoproof` on <decl>"). A flag
converts a fail-now into a pass-with-deferred-action.
