# ADR-002: Workout Versioning and Workout Session Snapshot Semantics

## Status

Accepted.

## Context

Two related open questions blocked the Training Planning / Training Execution boundary from stabilizing:

1. Is a Workout Session tied to a snapshot of the plan, or a live reference — i.e. if a Workout is edited, do in-progress or historical Sessions reflect the change? (`../../../scratchpad/open-questions/iteration-1.md` #2)
2. Does editing a Workout retroactively change what a Routine "means" for schedule occurrences already passed or in progress? (same file, #3)

The Architect's original proposal (`bounded-contexts.md`, Context 3) suggested Workout Session capture a denormalized snapshot of planned parameters at start time, but this was explicitly flagged by the Challenger as a product decision presented too close to a default — it has real product consequences (a user editing a Workout won't retroactively change what old or in-progress sessions show) that needed validation, not an architecturally-convenient default.

This decision spans two bounded contexts (Training Planning and Training Execution) and changes the shape of the Workout aggregate itself, so per the Context Promotion Rules it is recorded here rather than locally.

## Decision

**Workout is a versioned entity.** Editing a Workout's structure or parameters creates a new immutable version; it does not mutate the previous version in place. Prior versions remain intact and addressable.

**Routine references a Workout live (tracks latest).** A Routine's scheduled entry points at a Workout generically, not at a pinned version. It always resolves to whatever the current/latest version is at the moment it matters.

**Workout Session pins a specific Workout version.** When a session is started, it captures a reference to (or a copy derived from) the specific Workout version that was current at that moment. That reference never changes afterward, regardless of later edits to the Workout.

Net effect: editing a Workout affects all *future* Routine occurrences and any *new* Session started after the edit, but never changes the meaning of a Session that already started, and never rewrites history.

## Alternatives considered

- **Denormalized copy only, no versioning**: Workout stays a single mutable record; Workout Session copies the relevant planned values at start time with no reference back to Workout at all. Simpler to implement, but leaves question #3 (Routine's historical meaning) unresolved as a separate mechanism, and discards the ability to inspect "what did this Workout look like on date X" as a first-class capability. Rejected in favor of the single versioning mechanism, which resolves both questions #2 and #3 at once.
- **Fully live reference (no snapshot at all)**: rejected outright by the Analyst's requirement (`CLAUDE.md`) that planned and actual execution be structurally distinct and that historical sessions must not silently change meaning.

## Consequences

- **Training Planning** (`bounded-contexts.md`, Context 2): `Workout` aggregate must support version history, not just current-state mutation.
- **Training Execution** (`bounded-contexts.md`, Context 3): `WorkoutSession` aggregate references a specific immutable `Workout` version rather than the live `Workout` id.
- Implementation detail (whether the Session stores a foreign reference to a version row vs. a fully denormalized copy of that version's data) is left to the service/data-model layer — this ADR only fixes the conceptual semantics, not persistence shape.
- `Set` (see `domain-model.md`) is unaffected by this decision: it stays an inline ordered entry within whichever Workout version or Session it belongs to, not an independently-versioned concept.

## References

- `../../product/domain-model.md` → Workout, Routine, Workout Session
- `../bounded-contexts.md` (Context 2, Context 3)
- `../context-map.md`
- `../../../scratchpad/open-questions/iteration-1.md` (#2, #3)
- `../../../scratchpad/challenger-review-iteration-1.md`
