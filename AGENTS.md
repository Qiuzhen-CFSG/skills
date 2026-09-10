Focus on proving the intended mathematics. Use small theorem modules and concise task cards.
Git records versions; Lean and source review establish correctness.

## Scope and mathematical fidelity

- Work from repository evidence: declarations, sources, imports, usages, and
  diagnostics. Use `lean-search`, `lean-probe`, and `lean-proving`
  to resolve routine uncertainty.
- Establish the exact intended theorem before expensive proof work: compare its
  hypotheses, conclusion, namespace, instances, and public API with the source
  and actual consumers. Refer to existing declarations rather than copying their
  signatures into several tracking files.
- Do not silently weaken, strengthen, or replace a claim. Record evidence for a
  clear transcription repair. Ask when the mathematics or public API is genuinely
  ambiguous, required input is unavailable, or an action needs new authority.
- Audit suspicious source jumps: isolate the implication and hypotheses, try a
  smaller counterexample, then prove the missing transfer, repair a clear typo,
  or identify the source gap. Difficulty alone is not a source gap.
- Necessary upstream lemmas and their integration are in scope. Unrelated
  cleanup and wholesale library repartitioning are not.

## One theorem per proof task

- Each proof task names one target Lean file and one principal theorem to prove.
  Keep small private helpers with that theorem. Register a separate task when a
  needed lemma has a useful statement and can be proved independently.
- Prefer small modules. Consider decomposition above roughly 1500 lines. These are
  design signals, not quotas. Shared definitions, instances, and tightly coupled short
  results may form a cohesive module. Maintenance and source-audit tasks may have
  a different natural scope.
- Put reusable mathematics at the lowest coherent layer.
- Every module needs a `/-! ... -/` comment after its imports. Explain the result
  and essential hypotheses, its role in the development, the proof's mathematical
  idea, and source provenance. Substantial proofs need a sketch of the key reductions
  and imported results. Definitions and assembly modules explain their concepts or
  how the pieces combine. Keep this overview aligned with the actual proof; task
  logs belong in cards.
- Prefer existing APIs and explicit parameters. Reuse the exact action/invariance
  instance in dependent constructions; equivalent instances need not be
  definitionally equal.

## Scratch first, then integrate

1. Read the task card, target, relevant imports, and source. Develop the proof in
   `Scratch/<task-id>/` using the intended statement and production
   definitions. Keep experimental assumptions explicit; do not duplicate a
   definition to make a different statement easier to prove.
2. Check the scratch proof. If a prerequisite is missing, register its theorem
   task and continue useful independent work. Experiments conditional on that
   lemma remain in scratch until the actual lemma is proved.
3. Move the complete proof into the task's target module, preserving its public
   interface and updating the module overview. Integrate any required imports
   and wrappers in coordination with their agents.
4. Validate the production module and consumers before marking the task done.
   A successful scratch proof alone is not completion.

Do not add `sorry`, `admit`, `axiom`, or `opaque` to production source. Existing
placeholders are debt, not usable dependencies: prove required upstream results.
Disposable Lean sources belong in `Scratch/`, never `/tmp`; logs and other
non-source artifacts may use `/tmp`. Remove only your own scratch artifacts.

## Lightweight task state

- Each task has one Markdown card, normally `tasks/<id>.md`. Keep its theorem,
  target file, source, current route, scratch location, obstacle, next action,
  and validation results there. State only what another agent needs to resume.
- `tasks/index.md` holds campaign goals, important shared source findings, and
  authoritative build targets.
- `.agents/templates/` has minimal templates.
  Small one-turn maintenance edits need no task card.

On a cold start, read the task card. After compaction, continue
from the concrete next action. Update state when a route changes, a useful lemma
is isolated, a real blocker appears, or validation finishes. Keep durable failed
routes and source findings; skip routine searches, tactic transcripts, and
ceremonial logs. Existing long cards can be shortened when resumed without
losing useful proof evidence.

## Coordinating agents

- Within an authorized campaign, the coordinator may delegate bounded independent
  tasks. Agents can add needed theorem tasks themselves. Use an atomic task claim
  before dispatch so two agents do not take the same task.
- Prioritize prerequisites that unlock the current theorem and independent
  branches. Keep an explicit final assembly task so completed helpers lead back
  to the user's goal. Agent capacity is a resource limit, not a proof blocker.
- Coordinate directly before editing the same module or changing an interface
  another agent uses. Notify affected agents and update the relevant cards or
  dependencies when a lemma changes; use judgment about which proofs need
  rechecking. No formal contract-revision process is required.
- An assigned agent finishes or yields before another takes over. Silence alone
  does not establish that it stopped. Preserve useful partial proofs and leave
  a concrete next action at handoff.
- Inspect `git status --short` before editing. Preserve unrelated staged,
  unstaged, and untracked work. Never reset or discard changes for a clean base.
  Commit only when the user asks to commit or land; stage only authorized paths.
- Coordinate builds sharing writable Lake artifacts; use separate build trees
  for independent builds. Use Git diffs and rebuild affected modules after
  integration. Do not mistake an old `.olean` for validation of current source.

## Validation and persistence

- Prefer `lake build <Module>` for production validation; it applies project
  `leanOptions`. Use `lake env lean <file>` for focused scratch diagnostics.
  Save long output under `/tmp`, preserve exit status, and inspect with
  `rg -n 'error:|error\(|warning:|sorry|unsolved goals|timeout'`.
- Validate the owning module, relevant dependents, and the task's authoritative
  target. The final assembly owns campaign-wide validation. A root `lake build`
  covers only `defaultTargets`, not every library.
- Before completion: run `git diff --check`; compare placeholder matches with
  the pre-task baseline; rebuild before `#print axioms` and check every new or
  changed public theorem uses only `{propext, Classical.choice, Quot.sound}`;
  verify statements, names, visibility, imports, wrappers, module overviews,
  consumers, and production target coverage. Record commands and results in
  the card. Review the result separately from constructing the proof; another
  available agent can provide that review.
- A demonstrated pre-existing failure outside the task permits validation of
  the strongest unaffected target, with the exact command and first unrelated
  failure recorded. It never waives task-local checks or permits dependence on
  a placeholder.
- A failed route is local: record its decisive reason and try a structurally
  different route. Continue until the requested declaration cluster is proved
  and integrated, including helpers and final assembly.
- Stop the campaign incomplete only when all in-scope routes need an ambiguous
  or false statement changed, unavailable input/authority, or a reproducible
  environment failure that defeats safe alternatives. Report the exact blocked
  statement/command, evidence, routes ruled out, and smallest required input.
- Build in the repo env. Do not create new lakefile.toml which will leads to
  rebuild mathlib cache which is expensive.
