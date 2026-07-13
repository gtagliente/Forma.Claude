# ADR-006: Cross-Service Reference Integrity — Governed Point-to-Point Sync Calls

## Status

**Accepted** (2026-07-12, human product-owner sign-off, following Challenger review).

The Challenger's review (`../../../scratchpad/challenger-review-iteration-2.md`) endorsed the core decision (point-to-point over orchestrator) and raised one substantive refinement to the mechanism as drafted: scope fail-closed (Rule 2) to **shared** Exercises only, with fail-open-plus-warning for **private** ones, since a private Exercise's referencing Workouts always belong to the deleting user — there is no third party for the blanket block to protect. The product owner reviewed this and **explicitly chose to accept the ADR as originally drafted** (uniform fail-closed for all Exercise deletes, shared and private alike), not adopt the split. Recorded here so the refinement isn't lost — see "Alternatives considered" below and the Challenger review for the full argument, in case this is revisited later (e.g. if the availability cost of uniform fail-closed turns out to matter in practice).

## Context

Two features, built independently in `exercise-service` and `training-planning-service`, both depend on the same underlying capability — checking existence/reference of an ID across a service boundary — in opposite directions:

1. `training-planning-service`'s Workout create/edit references `Exercise` by ID, currently unvalidated (`../../services/training-planning-service/domain.md`).
2. `exercise-service`'s Exercise delete has no way to check whether a `Workout` still references the Exercise being deleted (`Forma.Exercise/docs/features/FT-003-update-delete.md` → "Central Architect Gate").

The Analyst (`../../product/requirements-and-open-items.md` → "Cross-context reference integrity") resolved the **business rules** for both: rule 1 is best-effort/UX-safeguard only (tolerates staleness — see glossary "Dangling Reference"); rule 2 is a hard block (extends the existing intra-service "can't delete an Exercise with hierarchy children" precedent across the service boundary), with an explicit asymmetry flagged: for rule 2, a false negative (check wrongly says "not referenced") is the dangerous failure mode; a false positive is a mere inconvenience.

This is not a one-off — per the Analyst, it is the general shape of any cross-service ID reference under [ADR-005](ADR-005-microservices-architecture.md), with `OwnerId`/`identity-service` as the near-certain next occurrence and intra-service Workout↔Routine deletion as an imminent same-shape (but not cross-service) instance.

ADR-005 explicitly deferred this exact question ("Inter-service integration pattern... belongs in `../integration-patterns.md`, currently empty"). Full rationale, options considered, and the current/near-term dependency graph this decision is evaluated against are in `../integration-patterns.md` — this ADR records the decision itself.

## Decision

Both drivers are resolved with **direct, synchronous, point-to-point read calls between `exercise-service` and `training-planning-service`** — no dedicated orchestration service, no async eventing/local read-model infrastructure adopted at this time. Each direction gets its own explicit failure-mode policy, derived from the business rule it serves, rather than one uniform mechanism applied to both:

- **Rule 1** (`training-planning-service` → `exercise-service`, at Workout create/edit): synchronous batch existence check, run inline in the same request. **Fails open** — if the check is inconclusive (timeout, service unreachable), the save proceeds anyway; only a *confirmed* negative result blocks the save with a clear message. The same capability, reused at read time, resolves Dangling References gracefully wherever a Workout is later shown (same fail-open policy).
- **Rule 2** (`exercise-service` → `training-planning-service`, at Exercise delete): synchronous reference-exists check, run inline in the same request. **Fails closed** — if the check is inconclusive, the delete is blocked; only a *confirmed* "not referenced" result allows it to proceed.

Each service exposes a narrow, single-purpose read-only endpoint for exactly what the other side needs (existence/reference lookup), not a general-purpose query API.

## Alternatives considered

- **Dedicated orchestration service** — rejected. Owns no domain data, so it fits no bounded context (unlike all four existing/proposed services, each 1:1 with a context per `../context-map.md`); doesn't reduce today's actual edge count (one bidirectional pair); not justified by any current scale/fan-out requirement.
- **Async events + local read-model cache** — deferred, not adopted. No cross-service messaging infrastructure exists in either codebase today; standing it up is a real cost that only pays off with a second independent consumer (the likely next one, Identity/`OwnerId`, isn't real yet). Also the wrong tool specifically for rule 2 — an eventually-consistent cache reintroduces the staleness risk rule 2's asymmetry can't tolerate.
- **One uniform mechanism for both rules** — rejected. Rule 1 and rule 2 have opposite risk profiles (tolerates staleness vs. cannot tolerate a false negative); forcing the same mechanism/failure-mode onto both would either make rule 1 unnecessarily brittle (fail-closed on a low-stakes check) or make rule 2 unsafe (fail-open on a high-stakes one).
- **Splitting Rule 2's failure mode by ownership** (fail-closed for shared Exercises, fail-open-with-warning for private ones) — raised by the Challenger, considered, and explicitly not adopted at human product-owner sign-off. Uniform fail-closed was accepted instead, for simplicity, with the trade-off (Exercise deletion unavailable for *all* Exercises during a `training-planning-service` outage, not just shared ones) knowingly accepted rather than unexamined. Revisit if this trade-off proves costly in practice.

Full comparative analysis: `../integration-patterns.md`.

## Consequences

- `exercise-service` and `training-planning-service` each take a new runtime dependency on the other, in each check's respective direction — a deliberate, narrow coupling (two single-purpose endpoints), not a general one.
- Exercise deletion becomes unavailable (by design — fail-closed) whenever `training-planning-service` can't confirm no Workout references the Exercise, including when `training-planning-service` itself is down. Accepted trade-off given the correctness requirement; flagged for the Challenger to stress-test.
- Workout create/edit stays available even when `exercise-service` is unreachable (by design — fail-open); the residual risk is exactly the narrow race window rule 1 already accepts as tolerable.
- A new nuance surfaced while designing rule 2's check, not resolved by this ADR: does "referenced" mean only a Workout's **current** version, or any historical version too? Provisional default (current-version-only) recorded in `../integration-patterns.md`; not a business-rule decision this ADR is authorized to make — routed back to the Analyst/product owner.
- No new decision is made here for `identity-service`/`OwnerId` references or the `training-planning-service` ↔ `training-execution-service` session-start data fetch — both explicitly out of scope, revisited when those services are further along.

## Explicitly deferred (not addressed by this ADR)

- Concrete API contracts (routes, payload shapes) — service-loop detail.
- Transport/technology choice (REST vs. gRPC, etc.) — same deferral as ADR-005.
- The Identity/`OwnerId` fan-out pattern, once `identity-service` is real.
- Whether "referenced" (rule 2) means current-version-only or any-version — flagged for Analyst confirmation.

## References

- `../integration-patterns.md` (full rationale and options analysis)
- `../../product/requirements-and-open-items.md` → "Cross-context reference integrity (Iteration 2)"
- `../../product/glossary.md` → "Dangling Reference"
- `ADR-005-microservices-architecture.md`
- `../../services/exercise-service/open-questions.md`, `../../services/training-planning-service/open-questions.md`
- `Forma.Exercise/docs/features/FT-003-update-delete.md`
