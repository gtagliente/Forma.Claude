# docs/agents/

## Purpose

Defines the AI agents that operate on this project's knowledge base: their responsibilities, expected inputs/outputs, and what parts of `docs/` each is allowed to update.

## What belongs here

- One `*-context.md` file per agent role (currently: `analyst-context.md`, `architect-context.md`, `challenger-context.md`).
- Each file is process/role documentation, not project knowledge itself — it describes *how an agent should behave*, not *what the domain or architecture is*.

## What does NOT belong here

- The actual output of an agent's work (domain analysis, architecture proposals, etc.) → their respective folders (`../product/`, `../architecture/`, `../../scratchpad/`).

## When an agent should load this context

- Every agent should load its own `<role>-context.md` at the start of a session to (re)confirm scope and boundaries.
- Any agent orchestrating a multi-agent iteration should load all three to know what to expect from/hand off to each role.

## Current state

Three roles defined: Analyst, Architect, Challenger — matching the iterative process described in `CLAUDE.md`. A fourth role (`development-agent-context.md`, for implementation work) is referenced in `CLAUDE.md`'s target structure but intentionally not created yet — no engineering conventions exist for it to follow *here*; implementation work itself now happens in each service's own repo (e.g. `Forma.Exercise`), which defines its own local pipeline of roles (Service Analyst, Service Architect, Backend Developer) in its own `docs/agents/`.

The Architect's scope has grown beyond the original one-time analysis phase: now that service repos exist and are being built, it also performs an ongoing **cross-service change review** — the final gate every service-local pipeline hands off to before merging. See `architect-context.md`.
