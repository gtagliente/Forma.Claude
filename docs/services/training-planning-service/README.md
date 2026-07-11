# training-planning-service

## What it is

Owns Training Planning: Workout (versioned — editing creates a new immutable version, [ADR-002](../../architecture/adr/ADR-002-workout-versioning-and-session-snapshot.md)) and Routine (references a Workout live, always the latest version). References Exercise by identity only, no duplication.

## Source

`bounded-contexts.md` (Context 2, Training Planning) → `context-map.md` → [ADR-005](../../architecture/adr/ADR-005-microservices-architecture.md) (service split, independent datastore).

## Status

Placeholder only. `domain.md`, `architecture.md`, `api-contracts.md`, `decisions/`, `open-questions.md` are not populated yet — pending the deferred integration-pattern and technology-stack decisions (see `../README.md`). Note for whoever designs the API contract: `training-execution-service` needs to read Workout Version data at session-start time (see ADR-005's consequence for ADR-002) — this is the first concrete cross-service call this project will need to design.
