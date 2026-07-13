# web-client

## What it is

The user-facing web application (React SPA). Unlike the other four entries in this folder, it owns no domain data and is not one of ADR-005's bounded contexts — it's a **consumer** that composes calls to `exercise-service`, `training-planning-service`, and (later) `training-execution-service`/`identity-service` into the screens a user actually sees (routines, workouts, exercises).

Per explicit product direction: all business/domain logic stays in the backend services. This app owns presentation (layout, graphics, navigation) and the wiring that calls backend APIs and maps their data into the existing UI — nothing else. If a request would require the client to decide something domain-shaped (e.g. what counts as a valid Workout), that belongs to the owning service, not here.

## Source

Not derived from `bounded-contexts.md` like the other four (it isn't a bounded context). Exists because the product needs an actual client; scope is "whatever the backend services expose, presented through the UI that already exists."

## Status

FT-001 (authentication, against the now-real `identity-service`/`Forma.Auth`) is the first feature moving through `Workout_React`'s own pipeline — see `Workout_React/docs/features/FT-001-auth.md`. The React app (Vite + TS + React Router) already has UI built against mock/local data — `RoutineCard`, `WorkoutCard`, `WorkoutList`, `ExerciseItem`, `ExerciseForm` components and `RoutineDetail`/`WorkoutDetails` pages — which FT-001 and the features after it (exercise/workout/routine CRUD+search) wire to real backends without changing their layout. See `open-questions.md` for what's still open.

`domain.md`/`architecture.md` are intentionally not created — this app has no owned domain slice, and its wiring pattern (API client shape, data-fetching approach) will be decided by the first feature's design rather than speculated up front.

## RepositoryPath

../../../../Forma.Resource/Workout_React

See `Workout_React/CLAUDE.md` for that repo's own entry point. Note: `Workout_React` is a subfolder of the shared `Forma.Resource` git repo, alongside unrelated sibling folders `Forma.Planner` (a separate service, `training-planning-service`) and `Template_DDD` — only `Workout_React/` is this app.
