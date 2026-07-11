# training-execution-service

## What it is

Owns Training Execution: Workout Session (the record of actual performance, offline-capable per [ADR-003](../../architecture/adr/ADR-003-offline-workout-session-logging.md)), and Progress Analytics as an internal capability, subject to the non-retroactive consistency model in [ADR-004](../../architecture/adr/ADR-004-progress-tracking-not-retroactively-recomputed.md).

A session pins a denormalized copy of the Workout Version it started against, rather than a cross-service reference resolved at read time — see [ADR-005](../../architecture/adr/ADR-005-microservices-architecture.md)'s consequence for ADR-002. May have attached Media Resources.

## Source

`bounded-contexts.md` (Context 3, Training Execution; Context 4, Progress Analytics) → `context-map.md` → ADR-005 (service split, independent datastore).

## Status

Placeholder only. `domain.md`, `architecture.md`, `api-contracts.md`, `decisions/`, `open-questions.md` are not populated yet — pending the deferred integration-pattern and technology-stack decisions (see `../README.md`).
