---
name: lean-search
description: Discover exact Lean 4 declarations, definitions, imports, and usage patterns in this repository or vendored Mathlib. Use for read-only API lookup and candidate search; use lean-probing to confirm a candidate under exact imports or diagnose elaboration, and lean-proving when implementing a proof.
---

# Lean search

Return an actionable result: the best declaration, its exact type and import,
how it matches the target, and any remaining mismatch. Search before recreating
infrastructure.

## Search from shape, not a guessed name

Extract the operation, important type constructors, namespace, relation
direction, and decisive hypotheses. Search several stable fragments and both
declarations and call sites; nearby uses often reveal argument order, instances,
coercions, and the intended import faster than a definition alone.

```bash
rg -n 'normalClosure|map_normalClosure' --glob '*.lean' --glob '!.lake/**' .
rg -n '^(public )?(theorem|lemma|def|abbrev|structure|class).*normalClosure' \
  --glob '*.lean' --glob '!.lake/**' .
```

Prefer a project wrapper when it expresses the exact local abstraction. Then
search vendored Mathlib, normally under `.lake/packages/mathlib/Mathlib`; if the
layout differs, locate it with `rg --files .lake/packages`.

```bash
rg -n 'map_normalClosure|normalClosure.*map' \
  .lake/packages/mathlib/Mathlib --glob '*.lean'
```

Read surrounding declarations, imports, and representative usages rather than
returning a name from an isolated matching line. Return the exact declaration,
its source/import path, how its type matches the target, and any mismatch.

## Leave elaboration checks to probing

Source presence does not by itself prove availability under the target's exact
imports or project options. If implicit arguments, coercions, typeclasses,
visibility, freshness, or a proof state need to be checked, hand the known
candidate to `lean-probing` instead of turning discovery into a production edit.

Use `@` in a later probe to expose implicit arguments; search results should
still preserve the declaration's ordinary type and representative call shape.

For type-directed search, try `exact?`, `apply?`, and `simp?`; use `aesop?` or
`grind` only when broader structural search fits the goal. Treat suggestions as
candidates: inspect their types and dependencies, then keep the generated tactic
only when it is stable and understandable.

When a candidate is close, check whether explicit specialization, symmetry,
`simpa`, `convert`, or one narrow bridge resolves the mismatch. Do not force a
theorem whose hypotheses or semantics differ.

## Negative results

If no declaration matches, report the searched fragments and locations, the
closest candidates, and the exact mismatch. Update a live task card only when
that negative result changes the proof route or identifies a genuinely missing
interface; do not log routine searches.
