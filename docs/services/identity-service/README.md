# identity-service

## What it is

Owns the `User` concept — the single normal-user persona confirmed by [ADR-001](../../architecture/adr/ADR-001-user-model-iteration-1.md). No coach/delegation, no roles, no account-state modeling beyond a minimal identity. Every other service scopes its data to a `User` owned here, referenced by ID.

## Source

`bounded-contexts.md` (Context 6, Identity) → `context-map.md` → [ADR-005](../../architecture/adr/ADR-005-microservices-architecture.md) (service split, independent datastore).

## Status

Placeholder only. `domain.md`, `architecture.md`, `api-contracts.md`, `decisions/`, `open-questions.md` are not populated yet — pending the deferred integration-pattern and technology-stack decisions (see `../README.md`).
