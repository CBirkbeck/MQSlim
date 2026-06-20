# Claude Context: MQSlim

## Project Purpose

MQSlim is the slim shadow of the [`mathlib-quality`](https://github.com/CBirkbeck/mathlib-quality)
Claude Code plugin. Same workflows, same goals — but written in the
"trust frontier-model judgement; minimal instructions" style Anthropic's
skill-authoring guide explicitly recommends.

The motivating observation (from external feedback): frontier models
comply over-eagerly with long instruction lists at the expense of
mathematical and engineering judgement. The hypothesis MQSlim tests:
restating the same goals in 1/5 the prose lets the model reason about the
specifics rather than execute a checklist.

## Current Status

**Version:** 0.51.0 (parallels mathlib-quality v0.51.0)

### Shipped commands (slim)

- `/clnup` — slim shadow of `/cleanup`
- `/mthlbl` — slim shadow of `/mathlibable`
- `/devel` — slim shadow of `/develop`
- `/bstmode` — slim shadow of `/beastmode`
- `/decoproof` — slim shadow of `/decompose-proof`

### To port (as mathlib-quality versions prove stable)

`/overvw` `/projstat` `/exprvw` `/genrlz` `/presub` `/fixfb` `/bmpml`
`/blprint` `/unform` `/contrib` `/intlrn` `/spltf` `/clnup-all`

## Naming convention

Slim commands use abbreviated names so they coexist with the full-fat
mathlib-quality plugin in the same session. User picks per task: type
`/cleanup` for the verbose, gated version; `/clnup` for the slim,
judgement-led version.

| Slim | Shadows | Why this abbreviation |
|------|---------|------------------------|
| `/clnup` | `/cleanup` | drop the vowel-heavy middle |
| `/mthlbl` | `/mathlibable` | drop most vowels |
| `/devel` | `/develop` | clip the suffix |
| `/bstmode` | `/beastmode` | clip the prefix |
| `/decoproof` | `/decompose-proof` | clip the suffix; preserve readability |

## Shadow-update contract (binding)

When mathlib-quality gains a new feature or refines an existing one,
MQSlim gets the **analogous slim restatement**. The bar:

1. Read the diff in mathlib-quality.
2. Identify the load-bearing intent — what is the new rule, what is the
   real failure mode it catches?
3. Restate in 1–3 sentences (or one short bullet list) in the MQSlim
   shadow file.
4. Link out to the full-fat reference for anyone who wants the verbose
   version — don't duplicate the long anti-pattern catalogues here.
5. Bump version to parallel mathlib-quality's.

The shadow is **not a literal copy at 1/5 length**. It's a re-derivation
of the same intent from first principles, in the slim style. Mechanical
truncation loses the load-bearing rails (the gates that catch real
failures) while keeping the over-specification (per-row required-artifact
tables, anti-pattern checklists). The slim style does the opposite:
keep the gates, drop the artifact pile-ons.

## Slim style: what we keep, what we drop

**KEEP** (the load-bearing rails — gates that catch real failure modes):
- The hard gates in `/clnup`: build clean, statement-protected, names follow
  convention, lines packed, inequalities oriented.
- The five verdict buckets in `/mthlbl` and the requirement that each cite
  evidence (the lit search, the mathlib search, the composition check).
- The pre-PR helper hygiene in `/decoproof` (the five inherited-context
  failure modes reviewers catch).
- `/bstmode`'s mandatory post-proof `/cleanup` call (gates the proof
  before mark-done).
- The Bourbaki 2.0 + concrete-via-abstract diagnostics — these are
  judgement-led patterns the model should know to apply.

**DROP** (the over-prescription):
- Per-rule status blocks ("for every rule in golfing-rules.md, print:
  applied / tried-and-failed / n/a"). Trust the model to apply rules and
  report concisely.
- Required-artifact tables enumerating every audit item. Replace with
  "report what you found, structured as you see fit, including the things
  the gates check".
- Anti-pattern catalogues with 12+ entries. The model knows what
  short-circuiting looks like; one or two canonical examples are enough.
- Verbose "Why this matters" / "Why we did it this way" prose. Frontier
  models infer this; we don't need to spell it out.
- Phase-numbered sub-step subdivisions (Phase 4.5, Step 2.6c, etc.).
  Replace with one numbered list at the top + the model arranges its work.

**ALSO DROP** (style we never wanted but accumulated):
- Citations to named reviewers / specific PR numbers in the skill content
  itself (those belong in the contribution PR's body, not the durable
  skill files). The full mathlib-quality plugin has been scrubbed; MQSlim
  ships without them from the start.

## Reference docs (slim)

Two reference files ship:

- `skills/mqslim/references/cleanup-gates.md` — the four hard gates
  (~100 lines vs the full plugin's 300).
- `skills/mqslim/references/mathlibable-buckets.md` — the five verdict
  buckets and what evidence each needs (~150 lines vs the full plugin's
  400+).

Workers in this skill should reach for these when they need the rails;
for everything else, they reason from the SKILL.md + their own
mathematical and engineering judgement.

## Testing the hypothesis

The point of MQSlim is to test whether the slim style produces better
outputs than the verbose style. The way we'll know:

- Run `/clnup` and `/cleanup` on the same file independently. Compare
  the outputs.
- Are the load-bearing fixes captured by both? (If `/clnup` misses
  something `/cleanup` catches, that's a real signal — refine the slim
  rails to include it.)
- Is `/clnup`'s output more readable? Are its judgement calls better-
  justified or worse?
- Does `/clnup` use less context? (Should be ~5× less.)

The result of this comparison should inform what the full mathlib-quality
plugin *actually needs* in its prompts vs. what's accumulated over-
specification. The slim version isn't just a smaller copy — it's a
probe into what's load-bearing.
