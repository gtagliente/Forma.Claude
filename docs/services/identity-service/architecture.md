# identity-service — Architecture (Implementation-Relevant Detail)

_Central-loop output (Architect), reconciled against what's actually implemented in `Forma.Auth`. Naming, exact schema, and endpoint-level detail are the Service Architect's job, in `Forma.Auth/docs/architecture/`._

## Technology shape — different from the other two services, deliberately

`identity-service` is Python/FastAPI, built on the [`fastapi-users`](https://fastapi-users.github.io/fastapi-users/) library, not .NET/Clean-Architecture/CQRS like `exercise-service` and `training-planning-service`. This is a legitimate per-service technology choice under [ADR-005](../../architecture/adr/ADR-005-microservices-architecture.md)'s independent-service model, not a convention this service failed to follow. Persistence is SQLite (`sqlite+aiosqlite`) — its own independent datastore, same rule as the other two, just lighter-weight infrastructure.

## API surface — already built via the library

Registration, JWT login/logout, password reset, email verification, and user self-service all exist today as `fastapi-users` router factories (see `Forma.Auth/docs/architecture/codebase-baseline.md`). REST contract: `Forma.Auth/openapi.json`. Nothing here needed a from-scratch API design pass — the Service Architect's job for this service's first features is more likely to be "does this fit the library's extension points (a `UserManager` hook, a schema field) or does it need genuinely new code," not "design an endpoint."

## Cross-service integration this service must design for

No other service validates a JWT issued here yet — `exercise-service` and `training-planning-service` both still use a caller-supplied `OwnerId`/`RequestingUserId` stand-in in place of real auth (see their own `domain.md`s' "Auth stand-in" notes). This service now existing and working is what makes that gap concrete rather than theoretical.

This is a **different** cross-service question from the one `../../architecture/integration-patterns.md`/[ADR-006](../../architecture/adr/ADR-006-cross-service-reference-integrity.md) already resolved (Accepted) — that document governs Exercise↔Workout existence/reference checks specifically. It explicitly named the Identity/`OwnerId` case as a related-but-separate, deliberately deferred future instance ("`identity-service` isn't real yet... Revisit once both are true," i.e. this service is real *and* there are at least two independent consumers needing the same capability), and predicted it would run at far higher call volume (every create, in every service) than either of ADR-006's two current drivers — a strong hint it may end up needing the async/local-cache option ADR-006 deferred, not necessarily the same synchronous point-to-point shape. This service being real now satisfies half that trigger. Don't invent an answer locally — this needs its own cross-service design pass, most naturally as a follow-up to `integration-patterns.md` once someone picks up the "wire real auth into a consuming service" feature.

## Technical setup notes (flag, not fix — from this pass's reconnaissance)

See `Forma.Auth/docs/architecture/codebase-baseline.md` for the full list (dead `Post` relationship, unused tutorial dependencies, hardcoded JWT/token secret, `.env`/`test.db` not gitignored, no email delivery, no migration tooling, no test suite at all). The one item worth central visibility, not just local tracking: the **hardcoded JWT secret** is a real security gap the moment any other service starts trusting tokens issued here — flagged in `open-questions.md` #2 as something to resolve before any cross-service auth wiring happens, not after.

## Status

First pass, reconciled against existing code. Supersedes "placeholder only" in `README.md`.
