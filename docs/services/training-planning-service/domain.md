# training-planning-service — Domain (Implementation-Relevant Detail)

_Central-loop output (Analyst/Architect) for a **greenfield** service — unlike `exercise-service`'s equivalent doc, there is no existing domain code to reconcile against (confirmed via inspection of `Forma.Planner/src`: zero aggregates, commands, handlers, or controllers, just the generic scaffold). This is prescriptive guidance the Service Analyst/Architect should follow when the first feature goes through this service's own local pipeline (`Forma.Planner/.claude/agents/`), not a description of what's already correct. See `../../product/domain-model.md` for the full, system-wide version of everything below._

## Aggregate: Workout

A reusable training plan composed of exercises, describing *intended* structure — not itself a specific occurrence in time.

- **Versioned (decided, [ADR-002](../../architecture/adr/ADR-002-workout-versioning-and-session-snapshot.md))**: editing a Workout's structure or parameters creates a new immutable version; prior versions remain intact.
- Composed of Exercises with Workout-specific parameters (sequence, reps, duration, sets, weight, rest time, supersets/circuits) — Exercise is referenced by identity only (owned by `exercise-service`), never duplicated.
- **Set** is an inline ordered entry within a Workout version — no independent identity, not addressable outside its parent (decided centrally).
- **Still open centrally, not this service's to decide alone**: can the same Exercise appear more than once in a Workout (e.g. warm-up set at lower weight, then working sets)? Is rest time per-exercise or per-set?

## Aggregate: Routine

Organizes Workouts over time — which workouts happen, when, how often.

- Must **reference** Workouts, not duplicate their detail.
- **Reference semantics (decided, ADR-002)**: references a Workout **live** — always the latest version, not pinned at creation time.
- **Still open centrally**: is a Routine's schedule a repeating pattern, bound to calendar dates, or both? Can rest days be modeled explicitly?

## Now built: Workout create + versioning

`Workout` (aggregate root, `OwnerId` + `Name` + `CurrentVersionNumber`) with `WorkoutVersion` child entities (immutable, own table, FK to `Workout`) holding `WorkoutExerciseEntry` owned-collection entries (`ExerciseId`, `Sets`, `Reps`/`DurationSeconds`, `Weight`, `RestSeconds`, `Sequence`, `GroupId` for supersets) — went through `Forma.Planner`'s own feature pipeline (`Forma.Planner/docs/features/FT-001-workout-create.md`). Resolves the aggregate-boundary question `architecture.md` originally left open (Option A). `ExerciseId` references are unvalidated (no cross-service check exists yet, per `open-questions.md` #5).

## Now built: Workout new version

`Workout.AddNewVersion(...)` — appends an immutable `WorkoutVersion` (`VersionNumber = CurrentVersionNumber + 1`), bumps `CurrentVersionNumber`, caller must be the Workout's owner (403 otherwise). Went through `Forma.Planner/docs/features/FT-002-workout-new-version.md`. Surfaced a reusable fix: `IWriteOnlyRepository<TEntity,TKey>.MarkModified<TProperty>` (mark exactly one scalar dirty via `DbContext.Entry()`, without cascading into navigation collections) — needed whenever a root's own scalar changes alongside a separately-tracked new child.

## Now built: Routine create

`Routine` (aggregate root, `OwnerId` + `Name`) with `RoutineEntry` owned-collection entries (`WorkoutId`, `DayOfWeek?` — a deliberately minimal, provisional scheduling placeholder, not a resolution of `open-questions.md` #1 — `Sequence`). Unlike `ExerciseId` on Workout, `WorkoutId` references are **validated** (`IWorkoutReferenceChecker`) — Workout is owned by this same service, so existence + ownership can actually be checked. A Routine may reference the same Workout more than once. Went through `Forma.Planner/docs/features/FT-003-routine-create.md`.

All three features requested this pass (Workout create, Workout new version, Routine create) are now built.

## What this service still needs to build

1. Update/Delete for both Workout and Routine, following the same shape `exercise-service` used (`Forma.Exercise/docs/features/FT-003-update-delete.md`), once Create exists for each.
2. Read access to Workout version history (list all versions of a Workout, or fetch a specific past version) — not requested by any feature yet.

## What this service does NOT own

Exercise (`exercise-service` — referenced by identity only), Workout Session / Progress Tracking (`training-execution-service`), User/Identity (`identity-service`). References `Exercise` and `User` by ID only, per ADR-005's independent-datastore rule — never a cross-service join.

## Status

First real pass. Supersedes "placeholder only" in `README.md`.
