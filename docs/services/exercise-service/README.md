# exercise-service

## What it is

Owns the Exercise Library: Exercise (shared/curated + private per-user, visibility scoped by owner), the parent/child generalization-specialization hierarchy, attached Media Resources, and AI Enrichment as an internal capability (not its own service — see `context-map.md`, "Deferred domain areas").

## Source

`bounded-contexts.md` (Context 1, Exercise Library; Context 5, AI Enrichment) → `context-map.md` → [ADR-005](../../architecture/adr/ADR-005-microservices-architecture.md) (service split, independent datastore).

## Status

This file stays a pointer/summary — the real, actively-maintained knowledge base for this service now lives in its own repository (see `RepositoryPath`, below). `domain.md`, `architecture.md`, `api-contracts.md`, `decisions/`, `open-questions.md` are intentionally not populated here; their equivalents live there instead (`docs/product/domain-slice.md`, `docs/architecture/`, `docs/agents/`, etc.).

## RepositoryPath

../../../Forma.Exercise

See `Forma.Exercise/CLAUDE.md` for that repo's own entry point — it implements this service, carries its own scoped `docs/` (domain slice, architecture, engineering, agents pipeline, features, branches), and is where actual development happens.