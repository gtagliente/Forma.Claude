# exercise-service — Domain (Implementation-Relevant Detail)

_Central-loop output (Analyst/Architect), reconciled against what's actually implemented in `Forma.Exercise` as of this pass. This is the "high-level instructions/technical definitions" the service's own local pipeline (see `Forma.Exercise/.claude/agents/`) reads before starting feature work — see `../../product/domain-model.md` for the full, system-wide version of everything below._

## Aggregate: Exercise

**Confirmed correct as built.** `Exercise` is the aggregate root (`Forma.Domain.Entities.ExerciseAggregate.Exercise`), matching the central domain model's Exercise concept. Current shape: `Name` (unique per ownership scope), `Description`, `MuscleGroups` (multi-valued), `OwnerId`, `ParentId`, plus a child collection of resources. Construction is factory-based (`Exercise.Create`/`Update`/`Delete`) with domain-side uniqueness/hierarchy checks — good, keep this pattern for new mutations rather than exposing public setters. Full CRUD is now wired end-to-end (Create, Update, Delete, plus `SetParent`/`ClearParent`) — see `Forma.Exercise/docs/features/FT-003-update-delete.md`.

## Media Resource — implemented as `ExerciseResource`

The central domain model's **Media Resource** concept (a photo/video, uploaded or linked, attachable to an Exercise) is already built, correctly modeled as a **child entity** of the Exercise aggregate (`ExerciseResource`: `Title`, `Content`, `Link`, `Type` — `Video|Image|Text|Uri`), not a separate aggregate. Its creation is `internal` and enforced to only happen via `Exercise.AddResource(...)`, with an automated architecture test guarding that boundary — this is correct DDD aggregate design and should not change.

**Open naming question, not decided here**: the central ubiquitous language calls this concept "Media Resource"; the code calls it `ExerciseResource`. Whether to rename, or treat "Media Resource" as the business term and `ExerciseResource` as a fine implementation-level name for this aggregate's local slice of it, is a **service-loop decision** (Service Analyst/Architect), not resolved by this pass — see `open-questions.md`.

Note for later: the central domain model also allows Media Resource on a Workout Session — that's `training-execution-service`'s concern, a separate implementation, not something `exercise-service` needs to account for.

## What this service still needs to build (not yet implemented at all)

Confirmed via code search — none of the following exist in `Forma.Exercise` today:

- **Equipment, Tags, Difficulty** — none exist as fields. Still open *centrally* whether Equipment/Tags need their own identity or stay free attributes, and whether Difficulty is global or per-user (see `open-questions.md`) — this service shouldn't invent an answer, but should track the question.
- **Enrichment separation** — no enrichment-related fields or the promotion mechanism exist. Fully greenfield; depends on `AI Enrichment`'s external capability, which this service depends on one-directionally per `../../architecture/bounded-contexts.md` (Context 5).

## Now built: Ownership / visibility

`Exercise.OwnerId` (`Guid?`) — null means shared-library, non-null means private to that owner — went through `Forma.Exercise`'s own feature pipeline (`Forma.Exercise/docs/features/FT-001-ownership-visibility.md`). No cross-service impact (references `identity-service`'s `User` by ID only, no real auth wiring yet since `identity-service` isn't built — see that feature's `design.md` for the caller-supplied stand-in). Name uniqueness is scoped by ownership (shared names unique among themselves; each owner's private names unique among their own), not global as it was before.

## Now built: Exercise hierarchy

`Exercise.ParentId` (`ExerciseId?`, self-referencing) — went through `Forma.Exercise/docs/features/FT-002-exercise-hierarchy.md`. Child aggregate holds the parent's ID only, per this doc's earlier aggregate-boundary guidance. Cycle and self-parenting are prevented; cross-visibility (can a private Exercise specialize a shared one?) is left **permissive and unrestricted** as a provisional implementation default — the central open question (`open-questions.md` #6) is still unresolved, this doesn't answer it.

## What this service does NOT own

Workout, Routine, Workout Session, Progress Tracking, Set, User/Identity — owned by other services. `MuscleGroup` (already implemented as an enum + generic `StaticValueObjects` lookup mechanism) stays inside this service; if Movement Pattern is ever added as a similar controlled-vocabulary enrichment output, the same mechanism is a reasonable precedent.

## Status

First real pass. Supersedes "placeholder only" in `README.md`.
