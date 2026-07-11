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

Empty — no ADRs have been accepted yet. Iteration 1 produced only proposals (`../bounded-contexts.md`, `../architecture-approach.md`); nothing has been decided, so nothing has been promoted here yet.
