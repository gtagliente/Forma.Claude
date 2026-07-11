# docs/architecture/adr/

## Purpose

Global Architecture Decision Records — the durable log of significant, cross-cutting technical decisions and their rationale.

## What belongs here

One file per decision (`ADR-NNN-short-title.md`), each recording: context, the decision, alternatives considered, and consequences. Only decisions that are:

- irreversible or costly to reverse, **or**
- affect more than one service/feature, **or**
- touch security, deployment, system-wide architecture, or cross-service APIs/events.

## What does NOT belong here

- Local/reversible decisions scoped to one service or feature — those live in `../../services/<service>/decisions/` or `../../features/<feature>/decisions/` until (if ever) they get promoted here.
- Proposals that haven't been decided yet — those stay in `../` (e.g. `bounded-contexts.md`, `architecture-approach.md`) or `../../../scratchpad/` until accepted.

## When an agent should load this context

- Before proposing any new cross-cutting decision (to check for existing precedent or conflicts).
- The Architect, when a locally-scoped decision is being promoted to global scope.

## Current state

Two ADRs accepted, both promoted from Iteration 1 open questions once the product owner decided them:

- **ADR-001** — Single normal-user model for this iteration (no coach/delegation).
- **ADR-002** — Workout versioning and Workout Session snapshot semantics (Routine tracks latest, Session pins a specific version).

Remaining Iteration 1 output is still proposal-only (`../bounded-contexts.md`, `../architecture-approach.md`, `../context-map.md`) except where updated to reflect the two ADRs above.
