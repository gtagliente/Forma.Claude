# exercise-service

## What it is

Owns the Exercise Library: Exercise (shared/curated + private per-user, visibility scoped by owner), the parent/child generalization-specialization hierarchy, attached Media Resources, and AI Enrichment as an internal capability (not its own service — see `context-map.md`, "Deferred domain areas").

## Source

`bounded-contexts.md` (Context 1, Exercise Library; Context 5, AI Enrichment) → `context-map.md` → [ADR-005](../../architecture/adr/ADR-005-microservices-architecture.md) (service split, independent datastore).

## Status

Placeholder only. `domain.md`, `architecture.md`, `api-contracts.md`, `decisions/`, `open-questions.md` are not populated yet — pending the deferred integration-pattern and technology-stack decisions (see `../README.md`).

## RepositoryPath

../../../Forma.Exercise