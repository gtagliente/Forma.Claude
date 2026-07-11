# ADR-001: Single Normal-User Model for Iteration 1

## Status

Accepted.

## Context

`requirements-and-open-items.md` and `bounded-contexts.md` (Context 6, Identity) left open whether Forma supports only a solo athlete managing their own training, or also a coach/trainer directing another person's training. The Analyst deliberately refused to invent this persona, and the Challenger flagged it as the single highest-leverage unknown: it determines whether an Identity/delegation context exists at all, and affects data ownership across every other bounded context. See `../../../scratchpad/open-questions/iteration-1.md` (#1).

This is a cross-cutting decision — it shapes data ownership/scoping in Exercise Library, Training Planning, Training Execution, and Progress Analytics alike — so per the Context Promotion Rules in `CLAUDE.md` it belongs here rather than in a single service's local decisions.

## Decision

For this iteration, Forma has exactly one user persona: a **normal user** who manages and performs their own training. There is no coach/trainer persona, no delegation, no differentiated account roles or account states. Every account is a simple, equally-privileged normal user.

All domain data (Exercises, Workouts, Routines, Workout Sessions) is scoped to the single user who owns it, with no cross-user access model beyond the shared Exercise Library (see `domain-model.md` → Exercise).

Non-human/service account types (API integrations, premium tiers, etc.) were considered and explicitly deferred — no concrete requirement exists for them yet.

## Alternatives considered

- **Coach-athlete delegation model**: rejected for now. No concrete requirement described anywhere in `CLAUDE.md`; adding it now would mean designing assignment, visibility, and permission concepts against zero validated need, which is exactly the premature complexity `CLAUDE.md`'s "Avoid Premature Complexity" principle warns against.

## Consequences

- **Identity context** (`bounded-contexts.md`, Context 6) is confirmed to exist, but stays deliberately minimal: a `User` concept sufficient to own/scope data, with no roles, delegation, or permission model.
- Every other bounded context can assume single-owner data scoping; no context needs to model "who can see this."
- If a coach/delegation model is needed later, it is expected to require revisiting data-ownership assumptions across every context — this ADR does not attempt to design for that possibility in advance.

## References

- `../../product/requirements-and-open-items.md` → Users
- `../../product/vision.md`
- `../../../scratchpad/open-questions/iteration-1.md` (#1)
- `../bounded-contexts.md` (Context 6)
