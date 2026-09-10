---
name: lean-proving
description: Construct, repair, and verify Lean 4 proofs in this repository, including replacing placeholders and debugging elaboration failures. Use when the requested outcome is a proved declaration; use lean-search or lean-probing alone for read-only investigation, lean-performance for measured compilation optimization, and lean-proof-cleanup for a behavior-preserving refactor.
---

# Lean proving

Deliver the requested declaration proved in its owning module, with its intended
statement and public boundary preserved. Follow the repository `AGENTS.md`; use
a live task card when the work is multi-step or already tracked.

## Establish the contract

- Inspect the exact theorem, namespace, variables, imports, visibility, wrappers,
  downstream uses, and cited mathematical source.
- Translate the statement into plain mathematical inputs, hypotheses, and
  conclusion. Compare source prose, Lean, and wrappers before investing in a
  route.
- Resolve genuine statement drift first. Do not silently alter the mathematics
  to fit an available lemma.
- Establish a targeted build baseline when the file is already edited, has
  relevant warnings, or may depend on stale artifacts.

## Search–probe–edit–build

1. Use `lean-search` for broad API discovery and `lean-probing` for uncertain
   types, imports, instances, or diagnostics. Prefer a proved existing
   interface over duplicating infrastructure.
2. Choose the smallest proof architecture that exposes the mathematical reason.
   For a hard route, record a short sketch and its main risk before coding.
3. Keep theorem-local facts local/private. Place reusable helpers at the lowest
   coherent layer with actual consumers; do not split files mechanically.
4. Edit tracked source without temporary `sorry`, `admit`, `axiom`, or `opaque`.
   Use disposable scratch examples for top-down experiments.
5. Run `lake build <Owning.Module>`, inspect the complete first diagnostic, and
   revise the cause rather than the symptom.
6. Once the owner builds, validate the narrowest relevant dependents and then
   the task's authoritative target.

Use tactics according to the goal, not a fixed preference ladder. Small
`calc` blocks and named facts are valuable when they make the mathematical
dependency clear; automation is appropriate when its domain matches and its
behavior remains stable.

## Hard routes and source gaps

Prune easy normality, containment, equality, and impossible branches early.
Isolate the hard inference as the weakest useful lemma, make the main theorem
typecheck against that interface when helpful, and prove dependencies in the
order that gives the quickest feedback.

When an informal proof jumps to a stronger conclusion, audit the exact
implication before formalizing it:

1. list the available hypotheses and the claimed local conclusion;
2. test whether the implication is false in a smaller or degenerate setting;
3. search for the missing standard hypothesis or project interface;
4. prove a narrow transfer lemma, repair an unambiguous transcription error, or
   record the source gap and pursue an independent route.

Do not ask the user to choose among tactics or plausible helper decompositions.
Persist through failed routes, but record the decisive reason before abandoning
one so it is not silently repeated.

## Diagnose by failure class

- **Unknown declaration/import:** search usages and verify the narrowest import.
- **Unification/coercion:** inspect `#check @name`, orientation, coercions, and
  explicit parameters; use `simpa` or `convert` only after semantic comparison.
- **Typeclass/definitional equality:** reuse the caller's exact named instance,
  keep scopes narrow, and bridge pointwise behavior instead of rewriting large
  dependent structures.
- **Simplification:** use `simp?` diagnostically, then prefer focused lemmas when
  a broad simp set is fragile or expensive.
- **Visibility:** apply `lean-module-system` to distinguish name visibility,
  import transitivity, and exposed bodies; do not widen a boundary merely to
  close the current goal.
- **Freshness:** rebuild edited source before trusting an importing probe or
  `#print axioms`.

## Completion

Build the owning and relevant downstream targets, inspect warning/error output,
scan changed Lean source for forbidden placeholders against the pre-task
baseline, run `git diff --check`, and check axioms for every new or changed
public theorem after rebuilding. Confirm names, statements, imports, visibility,
wrappers, and task state. A scratch success or one closed branch is not
completion.
