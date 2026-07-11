# ADR-005: Microservices Architecture — Four Services

## Status

Accepted.

## Context

`architecture-approach.md` (Iteration 1, Architect output) evaluated three system-structure options against the bounded contexts in `bounded-contexts.md`:

- **Option A — Modular monolith**: recommended by the Architect and endorsed by the Challenger (`../../../scratchpad/challenger-review-iteration-1.md`), since no scale, independent-deployment-cadence, or independent-team-ownership requirement had been demonstrated at the time.
- **Option B — Microservices**: one service per bounded context.
- **Option C — Unstructured monolith**: rejected outright as strictly worse than Option A.

The product owner has now explicitly chosen **Option B**, ahead of any newly demonstrated scale/team-ownership need — a deliberate investment in service independence now rather than a response to a discovered requirement. This ADR supersedes Option A as this project's direction.

## Decision

Forma is built as **four independently deployable services**, each owning an **independent datastore** (no shared database, no cross-service joins or foreign keys — references between services are by ID only, resolved via API calls or denormalized copies, never a database-level join):

1. **`identity-service`** — Identity (`bounded-contexts.md`, Context 6): a minimal `User` concept. No coach/delegation/roles ([ADR-001](ADR-001-user-model-iteration-1.md)).
2. **`exercise-service`** — Exercise Library (Context 1): Exercise (shared/private ownership, parent/child hierarchy), attached Media Resources, and AI Enrichment as an internal capability (not its own service — per `../context-map.md`, it hasn't earned independent structure).
3. **`training-planning-service`** — Training Planning (Context 2): Workout (versioned, [ADR-002](ADR-002-workout-versioning-and-session-snapshot.md)), Routine.
4. **`training-execution-service`** — Training Execution (Context 3): Workout Session (offline-capable, [ADR-003](ADR-003-offline-workout-session-logging.md)), and Progress Analytics as an internal capability, subject to the non-retroactive consistency model in [ADR-004](ADR-004-progress-tracking-not-retroactively-recomputed.md).

No service is proposed for Progress Analytics or AI Enrichment independently — both remain internal capabilities of the service that already hosts them, consistent with `context-map.md`'s existing stance that neither has earned independent structure yet. That stance is unaffected by the monolith→microservices change; it was never about deployment unit, only about whether the *concept* has enough independent state/consumers to justify a boundary at all.

## Consequence for ADR-002: Workout Version pinning crosses a service boundary

ADR-002 left the Workout Session's version-pinning mechanism as an implementation detail: "a foreign reference to a version row vs. a fully denormalized copy of that version's data." With Workout living in `training-planning-service` and Workout Session in `training-execution-service`, each with its own datastore, a same-database foreign reference is no longer possible.

**Resolved here**: a Workout Session **denormalizes a copy** of the pinned Workout Version's relevant data (exercises, sets, reps, weight, rest, grouping) at the moment the session starts, rather than storing a cross-service reference resolved at read time.

This is reinforced by [ADR-003](ADR-003-offline-workout-session-logging.md): a session that starts offline cannot perform a synchronous call to `training-planning-service` to resolve a Workout Version at all. The client must already have (or fetch when connectivity allows, before going offline) the Workout Version data it needs at session-start time — and that's exactly the data that gets cached locally and synced later. A pure reference-by-ID would leave an offline-started session unable to resolve its own plan.

## Alternatives considered

- **Option A (modular monolith)**: the Architect/Challenger's iteration-1 recommendation, given no demonstrated need at the time. Explicitly superseded by this decision — not because the original analysis was wrong about the *evidence available then*, but because the product owner has chosen to invest in service independence ahead of that evidence.
- **Option C (unstructured monolith)**: rejected for the same reason as in `architecture-approach.md` — discards the bounded-context work for no benefit.
- **Shared database across services**: considered and rejected. Would keep deployment flexibility but forfeit most of the actual benefit of choosing microservices (true data ownership independence) while still paying the operational cost of multiple deployables — the "distributed monolith" anti-pattern.

## Explicitly deferred (not addressed by this ADR)

- Inter-service integration pattern (synchronous REST vs. async events vs. both) — belongs in `../integration-patterns.md`, currently empty; a follow-up decision.
- Concrete technology stack and per-service API contracts — same deferral already stated in `architecture-approach.md` for Option A, carried forward unchanged.

## References

- `../architecture-approach.md`
- `../bounded-contexts.md`
- `../context-map.md`
- `ADR-001-user-model-iteration-1.md`, `ADR-002-workout-versioning-and-session-snapshot.md`, `ADR-003-offline-workout-session-logging.md`, `ADR-004-progress-tracking-not-retroactively-recomputed.md`
- `../../../scratchpad/challenger-review-iteration-1.md`
