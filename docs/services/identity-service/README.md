# identity-service

## What it is

Owns the `User` concept — the single normal-user persona confirmed by [ADR-001](../../architecture/adr/ADR-001-user-model-iteration-1.md). No coach/delegation, no roles, no account-state modeling beyond a minimal identity. Every other service scopes its data to a `User` owned here, referenced by ID.

## Source

`bounded-contexts.md` (Context 6, Identity) → `context-map.md` → [ADR-005](../../architecture/adr/ADR-005-microservices-architecture.md) (service split, independent datastore).

## Status

`domain.md`, `architecture.md`, and `open-questions.md` are now populated — this is the central loop's output (Analyst/Architect, reconciled against what's actually built in `Forma.Auth`): domain/architecture guidance and consolidated open questions, treated as the high-level input `Forma.Auth`'s own local pipeline reads before doing feature-level work (see `Forma.Auth/.claude/agents/`). Unlike the other two services, this one already had real, functioning code (a FastAPI app built on the `fastapi-users` library) rather than a bare template or a greenfield scaffold — closer to `exercise-service`'s bootstrap than `training-planning-service`'s.

`api-contracts.md` and `decisions/` stay unpopulated — explicitly deferred, not overlooked. `../../architecture/integration-patterns.md`/[ADR-006](../../architecture/adr/ADR-006-cross-service-reference-integrity.md) are now **Accepted**, but only for the Exercise↔Training-Planning existence/reference-check pair — that document explicitly still defers the **Identity/`OwnerId` fan-out pattern** ("`identity-service` isn't real yet... Revisit once both are true") as a separate, unresolved case. This service becoming real (this pass) satisfies half that trigger; see `open-questions.md` #9. The REST contract as it exists today is available directly as `Forma.Auth/openapi.json`. Implementation-level detail (schema, endpoint specifics, local ADRs, engineering standards) belongs in `Forma.Auth`'s own `docs/`, not duplicated here.

## RepositoryPath

../../../../Forma.Resource/Forma.Auth

See `Forma.Auth/CLAUDE.md` for that repo's own entry point — it implements this service, carries its own scoped `docs/` (domain slice, architecture, engineering, agents pipeline, features, branches), and is where actual development happens.
