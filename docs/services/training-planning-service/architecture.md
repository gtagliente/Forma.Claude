# training-planning-service — Architecture (Aggregate-Level Guidance)

_Central-loop output (Architect), for a greenfield service — no existing code to ground this against, unlike `exercise-service`'s equivalent doc. Guidance at the aggregate-boundary level; naming, exact field types, and migration sequencing are the Service Architect's job, in `Forma.Planner/docs/architecture/`._

## Workout aggregate boundary — resolved (was open, see below)

**Decided, via `Forma.Planner`'s FT-001 (`Forma.Planner/docs/features/FT-001-workout-create.md`): Option A.** `Workout` is the sole aggregate root; `WorkoutVersion` is a child entity (own table, FK to `Workout.Id`, same shape as `exercise-service`'s `Exercise`/`ExerciseResource`), holding an ordered collection of `WorkoutExerciseEntry` (an owned type, no independent identity — matches the central "Set has no identity outside its parent" decision). `Workout.CurrentVersionNumber` is a scalar pointer to the latest version.

Rejected Option B (each version its own aggregate instance, mirroring `exercise-service`'s Exercise hierarchy) for the reason anticipated here originally: a `WorkoutVersion` has no meaning detached from its `Workout`, unlike hierarchy parent/child Exercises, which are each independently owned/visible.

## Routine

`Routine` is its own aggregate root: a schedule structure (open — see `domain.md`) plus a collection of `WorkoutId` references (or schedule-entries each referencing a `WorkoutId`). References `Workout` by ID only — resolving "the current version" happens by reading the `Workout` aggregate, not by embedding or caching version data in `Routine`.

## Cross-service integration this service must design for

`training-execution-service` needs to read Workout **Version** data at session-start time (a Workout Session pins the specific version current at that moment, per ADR-002). This is the **first concrete cross-service call** this service needs to support. The actual integration pattern (sync REST vs. async events vs. client-carried data) is still undecided — `../../architecture/integration-patterns.md` is empty. Don't invent an answer locally.

There's now a **second driver** for that same undecided integration-pattern question, from a different service: `exercise-service` can delete an Exercise (`Forma.Exercise/docs/features/FT-003-update-delete.md`) with no way to check whether a `Workout` here still references it — worth this service's Architect being aware of when the Workout↔Exercise reference is actually built, even though solving it isn't blocking Create.

## Technical setup notes (flag, not fix — from this pass's bootstrap)

1. **Stray leftover file**: `Forma.Planner/src/Forma.Planner.CoreInfrastructure/Abstractions/IExerciseWriteOnlyRepository.cs` — an `IExerciseWriteOnlyRepository<TEntity, TKey>` interface with an `IsUniqueExerciseResourceLinkAsync` method, clearly copied from `exercise-service`'s own scaffold at some earlier point (pre-dates this service's own domain work, which hasn't started) rather than genericized for this service. Should be deleted or replaced with a real `IWorkoutWriteOnlyRepository`-shaped interface once Workout's Create feature is designed — not fixed in this pass since it doesn't block anything yet.
2. **Package-pin bug** (`Microsoft.Extensions.DependencyInjection` pinned below what `MediatR 13` transitively needs) — already fixed as part of this bootstrap, same one-line fix `exercise-service` needed (see `Forma.Exercise/docs/architecture/adr/ADR-001-strongly-typed-exercise-id.md`'s "Consequences" for the same bug there).
3. **`.sln` references a nonexistent `Forma.UnitTests` project** — same as `exercise-service` had; build individual `.csproj` files instead of the `.sln` until someone fixes or removes that reference.

## Status

First pass, prescriptive (greenfield — no code to reconcile against). Supersedes "placeholder only" in `README.md`.
