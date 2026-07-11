# exercise-service

## What it is

Owns the Exercise Library: Exercise (shared/curated + private per-user, visibility scoped by owner), the parent/child generalization-specialization hierarchy, attached Media Resources, and AI Enrichment as an internal capability (not its own service — see `context-map.md`, "Deferred domain areas").

## Source

`bounded-contexts.md` (Context 1, Exercise Library; Context 5, AI Enrichment) → `context-map.md` → [ADR-005](../../architecture/adr/ADR-005-microservices-architecture.md) (service split, independent datastore).

## Status

`domain.md`, `architecture.md`, and `open-questions.md` are now populated — this is the central loop's output (Analyst/Architect, reconciled against what's actually built in `Forma.Exercise`): aggregate-level domain/architecture guidance and consolidated open questions, treated as the high-level input `Forma.Exercise`'s own local pipeline reads before doing feature-level work (see `Forma.Exercise/docs/agents/process.md`).

`api-contracts.md` and `decisions/` stay unpopulated — explicitly deferred, not overlooked, pending the still-undecided inter-service integration pattern (`../../architecture/integration-patterns.md`, currently empty) per ADR-005. Implementation-level detail (exact field types, EF mappings, local ADRs, engineering standards) belongs in `Forma.Exercise`'s own `docs/`, not duplicated here.

## RepositoryPath

../../../Forma.Exercise

See `Forma.Exercise/CLAUDE.md` for that repo's own entry point — it implements this service, carries its own scoped `docs/` (domain slice, architecture, engineering, agents pipeline, features, branches), and is where actual development happens.