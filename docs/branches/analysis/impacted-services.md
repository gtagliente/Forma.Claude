# Branch: analysis — Impacted Services

## Current state

Four service placeholders bootstrapped under `../../services/` ([ADR-005](../../architecture/adr/ADR-005-microservices-architecture.md)):

- **identity-service** — `User` concept ([ADR-001](../../architecture/adr/ADR-001-user-model-iteration-1.md)).
- **exercise-service** — Exercise Library: definition, ownership/visibility, hierarchy, attached Media Resources, AI Enrichment as an internal capability.
- **training-planning-service** — Workout (versioned, [ADR-002](../../architecture/adr/ADR-002-workout-versioning-and-session-snapshot.md)), Routine.
- **training-execution-service** — Workout Session (offline logging, [ADR-003](../../architecture/adr/ADR-003-offline-workout-session-logging.md)), Progress Analytics as an internal capability ([ADR-004](../../architecture/adr/ADR-004-progress-tracking-not-retroactively-recomputed.md)).

Each has only a placeholder `README.md` — no `domain.md`/`architecture.md`/`api-contracts.md`/`decisions/`/`open-questions.md` yet. None are implemented; this branch produces documentation only.

## Cross-service consequence to design next

`training-planning-service` and `training-execution-service` need their first real integration: a Workout Version pinned by a Workout Session is resolved as a denormalized copy captured at session-start, not a cross-service reference (ADR-005's consequence for ADR-002). The actual integration pattern (sync call vs. async event vs. client-carried data) is still undecided — see `pending-items.md`.
