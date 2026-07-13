# Forma — Context Map (Iteration 1)

_Synthesizes `bounded-contexts.md` (Architect) as challenged in `../../scratchpad/challenger-review-iteration-1.md`. The domain-area scoping (which concepts get independent structure) reflects the Challenger's narrowed, more conservative proposal and remains the shared understanding. The deployment-unit question (whether those areas become services or modules) is decided — see "Status," below._

## Domain areas

Three domain areas are treated as sufficiently justified to shape structure right now; two more are recognized but deliberately **not** given independent structure yet (see "Deferred domain areas" below), per the Challenger's over-engineering findings.

```
┌─────────────────────┐        ┌───────────────────────┐        ┌───────────────────────┐
│   Exercise Library   │──ref──▶│   Training Planning    │──perf─▶│   Training Execution    │
│ (Exercise, Equipment,│        │ (Workout, Routine)      │        │  (Workout Session,      │
│  Tags; enrichment as │        │                         │        │   incl. progress        │
│  internal sub-area)  │        │                         │        │   analysis capability)  │
└─────────────────────┘        └───────────────────────┘        └───────────────────────┘
```

- **Exercise Library** — the vocabulary of movements. Includes AI enrichment as an internal concern (one-directional: enrichment reads/annotates Exercises, never the reverse), not a separate area yet.
- **Training Planning** — turns exercises into intended plans (Workout) and schedules (Routine).
- **Training Execution** — records what actually happened (Workout Session), and, for now, also owns the progress-analysis capability computed from that data (rather than a separate area), since it has no independent state of its own yet.

## Identity (confirmed, minimal — [ADR-001](adr/ADR-001-user-model-iteration-1.md))

Every domain area above implicitly needs "a user" to scope data to. This is now confirmed: Forma has a single normal-user persona (no coach, no delegation, no roles) for this iteration. Identity is not drawn as a fourth box on the diagram above because it stays deliberately thin *conceptually* — a single `User` concept that the other three areas reference for scoping, not an area with its own workflows or state beyond that. Being conceptually thin doesn't mean undeployed, though: per [ADR-005](adr/ADR-005-microservices-architecture.md), Identity is one of the four independently deployed services (`identity-service`) with its own datastore, same as the other three. See `bounded-contexts.md` (Context 6).

## Deferred domain areas (recognized, not yet structured)

- **Progress Analytics as its own area** — revisit once it has state/consumers beyond a view over Training Execution data.
- **AI Enrichment as its own area** — revisit once the "external intelligence service" has concrete integration concerns (async jobs, retries, cost isolation) that outgrow living inside Exercise Library.

## Service boundaries (decided — [ADR-005](adr/ADR-005-microservices-architecture.md))

The three domain areas above, plus Identity, are each an independently deployable service with an independent datastore — `exercise-service`, `training-planning-service`, `training-execution-service`, `identity-service`. This was a deliberate product-owner choice to invest in service independence ahead of any demonstrated scale/deployment-cadence/team-ownership need (the analysis that would normally justify it, per `architecture-approach.md`'s original modular-monolith recommendation, doesn't currently show one — the decision was made anyway).

No service is proposed for Progress Analytics or AI Enrichment independently; both remain internal capabilities of the service that already hosts them (Training Execution and Exercise Library respectively) — that call is about the *concept's* independence, not the deployment unit, and is unaffected by this decision.

Independent datastores mean no cross-service joins or foreign keys — every arrow in "Relationships between concepts" below is now also a **service boundary**, resolved via API calls or denormalized copies, not database references. See ADR-005 for the concrete consequence this has on Training Planning → Training Execution specifically (Workout Version pinning). Inter-service integration pattern (sync vs. async) is decided for one concrete pair — see `integration-patterns.md`/[ADR-006](adr/ADR-006-cross-service-reference-integrity.md) (Accepted: direct synchronous calls, Exercise↔Training-Planning existence/reference checks). Not yet extended to every service pair — Identity/`OwnerId` fan-out in particular remains open, see `../services/identity-service/open-questions.md`.

## Relationships between concepts (summary)

```
Exercise Library  ──referenced by (identity only, no duplication)──▶  Training Planning
Training Planning ──pins a specific Workout version at session start (ADR-002), denormalized copy across the service boundary (ADR-005)──▶  Training Execution
Training Execution ──aggregated internally into──▶  Progress analysis (capability, not yet its own area)
```

See `../product/domain-model.md` for the underlying business-level relationships this map is built from.

## Status

Reflects Iteration 1 discussion including Challenger input, updated for five accepted decisions: the user/coach question ([ADR-001](adr/ADR-001-user-model-iteration-1.md)), the snapshot-vs-live-reference question ([ADR-002](adr/ADR-002-workout-versioning-and-session-snapshot.md)), offline session logging ([ADR-003](adr/ADR-003-offline-workout-session-logging.md)), Progress Tracking's consistency model ([ADR-004](adr/ADR-004-progress-tracking-not-retroactively-recomputed.md)), and — superseding the modular-monolith stance this file originally described — the microservices decision ([ADR-005](adr/ADR-005-microservices-architecture.md)). The four-domain-area structure (three plus Identity) is otherwise unchanged; what changed is the deployment unit each now maps to. All items previously tracked in `../../scratchpad/open-questions/iteration-1.md` are resolved or deliberately deferred.
