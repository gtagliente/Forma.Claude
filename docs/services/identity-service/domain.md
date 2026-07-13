# identity-service — Domain (Implementation-Relevant Detail)

_Central-loop output (Analyst/Architect), reconciled against what's actually implemented in `Forma.Auth` as of this pass — closer to `exercise-service`'s situation (real, functioning code to reconcile against) than `training-planning-service`'s (greenfield). This is the "high-level instructions/technical definitions" the service's own local pipeline (see `Forma.Auth/.claude/agents/`) reads before starting feature work — see `../../product/domain-model.md` for the full, system-wide version of everything below._

## User

**Confirmed correct as built, for what exists so far.** `User` (`app.db.User`, `SQLAlchemyBaseUserTableUUID` + `Base`) matches the central domain model's minimal single-persona concept ([ADR-001](../../architecture/adr/ADR-001-user-model-iteration-1.md)): `id` (UUID), `email`, `hashed_password`, `is_active`, `is_verified` — no coach/delegation, no custom roles beyond the library's own `is_superuser` boolean (see "Tension with ADR-001" below). No Forma-specific profile fields (display name, body metrics, goals) exist yet — matches the central domain model's own note that "account/profile details... remain undefined."

Unlike the two .NET services, this isn't built as a DDD aggregate with factory methods and domain events — it's a thin FastAPI service built almost entirely on the [`fastapi-users`](https://fastapi-users.github.io/fastapi-users/) library's ready-made user-management logic. That's a legitimate, deliberate difference in shape for this service, not a gap to close to match the other two.

## Auth mechanism — already built, not this pass's decision to make

`fastapi-users` v14 already provides, via router factories (see `Forma.Auth/docs/architecture/codebase-baseline.md` for full detail):

- Registration, JWT login/logout (Bearer transport, 1-hour token lifetime, no refresh token), password reset, email verification, user self-service (`/users/me`, superuser-gated `/users/{id}`).

This is real, working auth infrastructure — the central domain model's previously-undefined "account/profile details (auth...)" question is now substantially answered by this library's choices, not by a fresh design decision. The REST contract is available as `Forma.Auth/openapi.json`.

## Tension with ADR-001, not yet resolved

ADR-001 says "no differentiated account roles." The `fastapi-users` library bakes in an `is_superuser` boolean on every `User` regardless — likely intended as "administers this auth system," not a Forma-domain "coach" or "admin" role, but nothing in this codebase defines what `is_superuser` is actually *for* yet (currently it only gates the library's own `/users/{id}` routes). Flagged as an open question (`open-questions.md` #1) rather than resolved here — could turn out to be harmless (an auth-system-internal concern, out of ADR-001's scope) or could need an explicit central decision if a real "admin manages other users" capability is ever requested.

## What this service still needs to build (not yet implemented at all)

- **Any Forma-specific User fields** — display name, or anything else a Forma UI would want beyond bare email/password identity.
- **Real email delivery** for password reset / verification — currently the handlers only print the token to console (`Forma.Auth/docs/architecture/codebase-baseline.md`).
- **Externalized JWT/token secret** — currently a hardcoded literal in source, reused for JWT signing and both token-reset secrets (see `open-questions.md` #2 — a real security gap, not a style nit).
- **Cross-service auth wiring** — no other service (`exercise-service`, `training-planning-service`) actually validates a JWT issued here yet; each still uses a caller-supplied `OwnerId`/`RequestingUserId` stand-in (see their own `domain.md`s). This service existing doesn't retroactively wire that up — that's separate, future integration work.
- **Migration tooling** — schema changes currently apply via `Base.metadata.create_all` (create-if-missing), not a real migration system.

## What this service does NOT own

Exercise, Workout, Routine, Workout Session, Progress Tracking — owned by other services, each referencing `User` by ID only. This service does not know about any of those concepts.

## Status

First real pass. Supersedes "placeholder only" in `README.md`.
