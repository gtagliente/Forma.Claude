# Branch: analysis — Objectives

## Purpose

Carry out `CLAUDE.md`'s "Initial Task": perform a complete domain and architecture analysis for Forma before any production code is written. No feature implementation happens on this branch — its output is entirely the knowledge structure under `../../` (product, architecture, ADRs) and the scratchpad working notes.

## Scope

Following `CLAUDE.md`'s Analyst → Architect → Challenger → Knowledge Update cycle:

1. Produce an initial domain model, requirements/open-items list, glossary, and candidate bounded contexts from `CLAUDE.md`'s product description alone (Iteration 1).
2. Surface open questions the Analyst/Architect/Challenger pass can't resolve without human input, prioritized by leverage (`../../../scratchpad/open-questions/iteration-1.md`).
3. Route those open questions to the product owner and persist their decisions as ADRs / domain-model updates once made (the Knowledge Update step) — repeating until the open-questions backlog is empty or only contains deliberately deferred items.

## Definition of done for this branch

- Every Iteration 1 open question is either resolved (with an ADR where the decision is cross-cutting) or explicitly, deliberately deferred — not silently dropped.
- The domain model, glossary, bounded contexts, and context map are internally consistent and cross-reference each other and the ADRs correctly.
- No application code exists yet — that's out of scope until a future branch, once `CLAUDE.md`'s bar ("domain and architecture are sufficiently understood") is judged met.

## Current status

Iteration 1's open-questions list is fully closed out: high/medium-priority items 1–8 resolved via [ADR-001](../../architecture/adr/ADR-001-user-model-iteration-1.md)–[ADR-002](../../architecture/adr/ADR-002-workout-versioning-and-session-snapshot.md) and domain-model updates; lower-priority items 9–14 resolved via [ADR-003](../../architecture/adr/ADR-003-offline-workout-session-logging.md)/[ADR-004](../../architecture/adr/ADR-004-progress-tracking-not-retroactively-recomputed.md) or explicitly deferred (units/localization, notifications, monetization, body metrics/goals). The architecture-approach decision is also now made: [ADR-005](../../architecture/adr/ADR-005-microservices-architecture.md) — microservices, four services, each with an independent datastore, placeholder folders bootstrapped under `../../services/`. See `pending-items.md` for what's next.
