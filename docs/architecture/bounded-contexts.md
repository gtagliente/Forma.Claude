# Forma — Candidate Bounded Contexts (Iteration 1)

_Architect output. Derived **only** from `../product/domain-model.md` and `../product/requirements-and-open-items.md`. These are proposals for discussion, not decisions — nothing here is an ADR yet._

## Candidate contexts

### 1. Exercise Library

Owns the Exercise concept: definition, attributes (name, description, instructions, equipment, media, tags, difficulty), and its lifecycle independent of any plan.

- **Candidate aggregate**: `Exercise` (aggregate root). Equipment/Tags are likely value objects or references within it at this stage — not enough is known yet to justify their own aggregates (see Analyst's open item on whether Equipment needs independent identity).
- **Ownership (decided)**: an Exercise is either part of the shared/curated library (visible to all users) or a private Exercise owned by one user (visible only to them). Promotion from private to shared is deferred.
- **Hierarchy (decided)**: an Exercise may reference a parent Exercise (generalization/specialization), letting variants like "Barbell Bench Press" relate back to a general "Bench Press."
- Explicitly does **not** own enrichment content (see Context 5).

### 2. Training Planning

Owns Workout (exercises + intended parameters) and Routine (arrangement of Workouts over time). Grouped together because a Routine's core job — referencing Workouts without duplicating them — is an internal consistency concern within "planning," not a cross-context one.

- **Candidate aggregates**: `Workout` (root; contains ordered exercise entries with planned sets/reps/weight/rest/superset-circuit grouping, referencing Exercise by identity only) and `Routine` (root; contains scheduled references to Workouts).
- **Decided ([ADR-002](adr/ADR-002-workout-versioning-and-session-snapshot.md))**: `Workout` is versioned — editing creates a new immutable version rather than mutating in place. `Routine` holds a live reference to a Workout, always resolving to its latest version.

### 3. Training Execution

Owns Workout Session: the record of actual performance. Kept separate from Training Planning because `CLAUDE.md` is explicit that planned and actual execution are structurally distinct, and because this context has different consistency needs (append-mostly event-like data vs. editable plan templates).

- **Candidate aggregate**: `WorkoutSession` (root; contains actual per-set results). **Decided ([ADR-002](adr/ADR-002-workout-versioning-and-session-snapshot.md))**: the session references a specific immutable Workout Version, pinned at the moment the session starts — resolving the "Routine/Workout changed after the fact" ambiguity by construction.

### 4. Progress Analytics

Owns Progress Tracking: trends, PRs, training volume, consistency — derived from Training Execution history (and potentially future body-metrics data, per Analyst open items).

- Likely **not** an aggregate-owning context at all in the traditional sense — more a read/analysis model over Context 3's data. Whether it needs its own persistence (a materialized view) or can be computed on demand is an implementation question, not a boundary question, and is deferred.

### 5. AI Enrichment

Owns the enrichment workflow for Exercises (muscle groups, movement pattern, progressions/regressions, alternatives, common mistakes, safety notes) via an external/AI intelligence capability. Kept as its own context because `CLAUDE.md` explicitly mandates separation from the core domain.

- Depends on Context 1 (needs to know which Exercises exist) but Context 1 should not depend on it — the Exercise Library must remain coherent with zero enrichment data present.
- **Open**: whether enrichment output is auto-merged into what a user sees on an Exercise, or held as a separate, clearly-labeled suggestion pending acceptance (Analyst open item).

### 6. Identity (confirmed, deliberately minimal — [ADR-001](adr/ADR-001-user-model-iteration-1.md))

Owns a single `User` concept, sufficient to scope/own data across every other context. **Decided**: no coach↔athlete relationships, no delegated access, no roles — Forma has exactly one normal-user persona for this iteration. Kept intentionally thin; not proposed as a source of complexity.

## Cross-context relationships

```
Identity (minimal) ──owns/scopes data for──▶ all other contexts
Exercise Library ◀──enriches (one-way dependency)── AI Enrichment
Exercise Library ──referenced by (identity only)──▶ Training Planning
Training Planning ──performed as / pins a version of (ADR-002)──▶ Training Execution
Training Execution ──aggregated by──▶ Progress Analytics
```

No context currently appears to need a *synchronous, two-way* dependency on another — every arrow above is one-directional read/reference. This is relevant input for `architecture-approach.md`.

## Explicitly not decided here

- Whether these contexts become separate services, separate modules in one deployable, or something else — see `architecture-approach.md`.
- Exact aggregate field-level shape — this is a conceptual boundary proposal, not a data model.
