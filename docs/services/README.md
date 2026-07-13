# docs/services/

## Purpose

Holds per-service knowledge, once services actually exist. A service here means an independently deployable/ownable unit — this folder existing does **not** imply the project has committed to a microservices architecture (see `../architecture/architecture-approach.md` and `context-map.md` for that evaluation).

## What belongs here

One subfolder per service (e.g. `identity-service/`, `exercise-service/`), each containing:

- `README.md`, `context.md` — what the service is and why it exists.
- `domain.md` — the service's slice of the domain model, in implementation-relevant detail.
- `architecture.md` — internal architecture of the service.
- `api-contracts.md` — its external interface.
- `decisions/` — decisions local to this service.
- `open-questions.md` — unresolved items specific to this service.

Also (at this folder's root, once populated): `service-map.md`, `dependencies.md` — cross-service overview material that's specific enough to services (not the whole system) to live here rather than in `../architecture/`.

## What does NOT belong here

- Whether services should exist at all, and how many → `../architecture/` (proposal stage) then `../architecture/adr/` (once decided).
- Domain concepts shared across all services → `../product/`.

## When an agent should load this context

- Any agent working on one specific service — load only that service's subfolder.
- The Architect, when evaluating service boundaries or dependencies.

## Current state

Four services decided ([ADR-005](../architecture/adr/ADR-005-microservices-architecture.md)): `identity-service`, `exercise-service`, `training-planning-service`, `training-execution-service`. Three now have real implementations and populated `domain.md`/`architecture.md`/`open-questions.md`: `exercise-service` (`Forma.Exercise`), `training-planning-service` (`Forma.Planner`), and `identity-service` (`Forma.Auth`, Python/FastAPI via `fastapi-users` — a different technology stack from the other two, a deliberate per-service choice under ADR-005, not an inconsistency). `training-execution-service` is still a placeholder `README.md` only, with no implementation and nothing beyond it populated. All three implemented services still have `api-contracts.md`/`decisions/` unpopulated as formal artifacts in this folder — not because the integration pattern is undecided in general (`../architecture/integration-patterns.md`/[ADR-006](../architecture/adr/ADR-006-cross-service-reference-integrity.md) are Accepted for the Exercise↔Training-Planning existence/reference-check pair specifically), but because no service has had a pass dedicated to writing up its formal external contract yet. The Identity/`OwnerId` fan-out pattern (how other services would validate a token issued by `identity-service`) remains a distinct, still-open question — see `identity-service/open-questions.md` #9.

A fifth entry, `web-client/`, was added for the user-facing React app — it is **not** a fifth bounded-context service from ADR-005, just a consumer of the four above, documented here because it's still an independently deployable/ownable unit per this folder's definition. See `web-client/README.md` and `web-client/open-questions.md` (the latter records why it isn't wired to real backend calls yet).
