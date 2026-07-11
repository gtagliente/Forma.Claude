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

Empty. No services have been decided yet — the project is still evaluating whether/how to decompose Forma at all (see `../architecture/architecture-approach.md`). This folder is a placeholder for when that decision is made.
