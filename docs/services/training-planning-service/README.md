# training-planning-service

## What it is

Owns Training Planning: Workout (versioned — editing creates a new immutable version, [ADR-002](../../architecture/adr/ADR-002-workout-versioning-and-session-snapshot.md)) and Routine (references a Workout live, always the latest version). References Exercise by identity only, no duplication.

## Source

`bounded-contexts.md` (Context 2, Training Planning) → `context-map.md` → [ADR-005](../../architecture/adr/ADR-005-microservices-architecture.md) (service split, independent datastore).

## Status

`domain.md`, `architecture.md`, and `open-questions.md` are now populated — this is the central loop's output (Analyst/Architect) for a **greenfield** service (unlike `exercise-service`, there was no existing domain code to reconcile against): aggregate-level guidance and consolidated open questions, treated as the high-level input `Forma.Planner`'s own local pipeline reads before doing feature-level work (see `Forma.Planner/.claude/agents/`). The repository itself was renamed from the shared `Forma.Resource` template naming to `Forma.Planner` (solution, all projects) as part of this pass.

`api-contracts.md` and `decisions/` stay unpopulated as formal artifacts here — not because the integration pattern is undecided in general (`../../architecture/integration-patterns.md`/[ADR-006](../../architecture/adr/ADR-006-cross-service-reference-integrity.md) are Accepted for the Exercise↔Training-Planning existence/reference-check pair specifically), but because this service hasn't had a pass dedicated to writing up its formal external contract yet. Note for whoever does: `training-execution-service` needs to read Workout Version data at session-start time (see ADR-005's consequence for ADR-002) — a second, still-undesigned cross-service call distinct from ADR-006's two. Implementation-level detail belongs in `Forma.Planner`'s own `docs/`, not duplicated here.

## RepositoryPath

../../../../Forma.Resource/Forma.Planner