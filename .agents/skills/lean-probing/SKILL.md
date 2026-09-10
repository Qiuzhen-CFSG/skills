---
name: lean-probing
description: Confirm known Lean 4 declarations and diagnose types, proof states, imports, visibility, freshness, and axioms with disposable local probes. Use for focused read-only investigation after candidate discovery; use lean-search for broad lookup and lean-proving when implementing the final proof.
---

# Lean CLI probing

Use a small, reproducible probe to answer one uncertainty at a time. This skill
confirms or falsifies a known candidate; it does not choose the final API or
edit production proofs. Use `lean-search` for broad declaration discovery. Put
Lean probe sources in `Scratch/<descriptive-name>/`, not `/tmp`; logs may go under
`/tmp`. Preserve user-created scratch files and remove only artifacts created by
the current task.

## Probe loop

1. Import the narrowest module that should expose the declarations under study.
2. Reproduce only the relevant variables, instances, hypotheses, and goal.
3. Run `lake env lean Scratch/<name>/Probe.lean` from the repository root.
4. Read the full diagnostic, revise one uncertainty, and rerun.
5. Transfer the result to the owning source module and validate that module with
   `lake build <Target.Module>`.

A probe may contain an explicitly disposable `sorry` or an intentionally failing
example to display a goal. Never copy that placeholder into tracked source or
count probe elaboration as final validation.

## Inspect declarations and goals

```lean
#check Fully.Qualified.name
#check @Fully.Qualified.name
#print Fully.Qualified.name

set_option pp.all true in
#check Fully.Qualified.name
```

- `#check` gives the elaborated type; `@` exposes implicit arguments.
- `#print` reveals the available declaration body or statement.
- Scope `pp.all` narrowly so coercions, universes, synthesized arguments, and
  reducible wrappers are visible without flooding every diagnostic.

Use a minimal example and `trace_state` at the disputed step:

```lean
example {G : Type*} [Group G] (H : Subgroup G) : H ≤ H := by
  intro x hx
  trace_state
  exact hx
```

Test competing candidate terms separately. Preserve the exact mismatch from a
failure; do not retry the same term without a new reason.

## Freshness and authority

- Importing an edited module loads its last built `.olean`, not unbuilt source.
- Use `lake env lean path/to/Target.lean` for direct diagnostics, then rebuild the
  owner before inspecting exported declarations through an importing probe.
- Derive a build target by dropping `.lean` and replacing `/` with `.`.
- `lake build` is authoritative because it applies repository `leanOptions` that
  a direct Lean invocation may omit.

After rebuilding, use `#print axioms Fully.Qualified.name` for exported theorems.
Treat `sorryAx` as unresolved proof debt and compare the result with the
repository axiom allowlist.

When a failure appears only downstream, compare visibility, import
transitivity, the elaborated public type, and the exact instances before
rewriting the proof.
