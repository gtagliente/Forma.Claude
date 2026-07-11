# docs/branches/

## Purpose

Holds branch-scoped, *temporary* context: state that's only meaningful while a particular git branch is alive, and that shouldn't pollute the permanent product/architecture knowledge.

## What belongs here

One subfolder per long-lived or notable branch (e.g. `main/`, `develop/`, `feature-workout-builder/`), containing things like:

- `objectives.md` — what this branch is trying to achieve.
- `impacted-services.md` — what areas of the system it touches.
- `decisions/` — decisions made only for the lifetime of this branch.
- `pending-items.md` — work not yet finished on this branch.

## What does NOT belong here

- Anything meant to outlive the branch — once a branch merges, any durable decision must be promoted to `../architecture/adr/`, `../product/`, `../services/<service>/`, or `../features/<feature>/` before the branch context is considered closed. Branch context that never gets promoted is expected to become stale/discardable after merge.

## When an agent should load this context

- Any agent working directly on that branch, to pick up where a previous session left off.
- Not relevant to whole-product or whole-service analysis.

## Current state

One branch documented: `analysis/` — the current working branch, carrying out `CLAUDE.md`'s Initial Task (domain/architecture analysis) before any implementation. See `analysis/objectives.md`.
