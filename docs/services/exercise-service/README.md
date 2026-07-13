# exercise-service

## What it is

Owns the Exercise Library: Exercise (shared/curated + private per-user, visibility scoped by owner), the parent/child generalization-specialization hierarchy, attached Media Resources, and AI Enrichment as an internal capability (not its own service — see `context-map.md`, "Deferred domain areas").

## Source

`bounded-contexts.md` (Context 1, Exercise Library; Context 5, AI Enrichment) → `context-map.md` → [ADR-005](../../architecture/adr/ADR-005-microservices-architecture.md) (service split, independent datastore).

## Status

`domain.md`, `architecture.md`, and `open-questions.md` are now populated — this is the central loop's output (Analyst/Architect, reconciled against what's actually built in `Forma.Exercise`): aggregate-level domain/architecture guidance and consolidated open questions, treated as the high-level input `Forma.Exercise`'s own local pipeline reads before doing feature-level work (see `Forma.Exercise/.claude/agents/`).

`api-contracts.md` and `decisions/` stay unpopulated as formal artifacts here — not because the integration pattern is undecided in general (`../../architecture/integration-patterns.md`/[ADR-006](../../architecture/adr/ADR-006-cross-service-reference-integrity.md) are Accepted for the Exercise↔Training-Planning existence/reference-check pair specifically), but because this service hasn't had a pass dedicated to writing up its formal external contract yet. Implementation-level detail (exact field types, EF mappings, local ADRs, engineering standards) belongs in `Forma.Exercise`'s own `docs/`, not duplicated here.

## RepositoryPath

../../../../Forma.Exercise

See `Forma.Exercise/CLAUDE.md` for that repo's own entry point — it implements this service, carries its own scoped `docs/` (domain slice, architecture, engineering, agents pipeline, features, branches), and is where actual development happens.