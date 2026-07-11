# Branch: analysis — Impacted Services

## Current state

None. No service exists yet — `../../services/` is still an empty placeholder (see `../../services/README.md`), since the project hasn't decided whether/how Forma decomposes into services at all (`../../architecture/architecture-approach.md` currently recommends a modular monolith; not yet an ADR).

## Conceptual areas shaped by this branch

This branch only produces documentation, but the domain/architecture analysis it produced *shapes* what will eventually become service or module boundaries, per `../../architecture/context-map.md`:

- **Exercise Library** — Exercise definition, ownership/visibility, hierarchy, attached Media Resources.
- **Training Planning** — Workout (versioned), Routine.
- **Training Execution** — Workout Session, offline logging, Progress Analytics (folded in as a capability, not yet its own area).
- **Identity** — deliberately minimal (single `User` concept, no coach/delegation).

None of these have a corresponding folder under `../../services/` yet — they remain conceptual bounded contexts in `../../architecture/bounded-contexts.md` until (if ever) service extraction is justified.
