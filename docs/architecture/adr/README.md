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

Six ADRs accepted, all promoted from Iteration 1/2 open questions and follow-on architecture decisions once the product owner decided them:

- **ADR-001** — Single normal-user model for this iteration (no coach/delegation).
- **ADR-002** — Workout versioning and Workout Session snapshot semantics (Routine tracks latest, Session pins a specific version).
- **ADR-003** — Offline-capable Workout Session logging (local cache, sync on reconnect; scoped to session logging only).
- **ADR-004** — Progress Tracking computations are not retroactively recomputed when a historical session is edited/deleted.
- **ADR-005** — Microservices architecture: four independently deployable services (identity, exercise, training-planning, training-execution), each with an independent datastore.
- **ADR-006** — Cross-service reference integrity: governed, direct point-to-point synchronous calls between `exercise-service` and `training-planning-service` (fail-open for the best-effort Exercise-existence check at Workout create/edit, fail-closed for the hard-block Workout-reference check at Exercise delete, uniform across shared and private Exercises). No orchestration service, no async eventing adopted at this time. Accepted 2026-07-12 — see `ADR-006-cross-service-reference-integrity.md` and `../integration-patterns.md`.

Remaining Iteration 1 output is still proposal-only (`../bounded-contexts.md`) except where updated to reflect the six accepted ADRs above. `../architecture-approach.md` and `../context-map.md` have been updated to reflect ADR-005's outcome. `../integration-patterns.md` (previously empty) is now populated with the accepted mechanism underlying ADR-006.

`../integration-patterns.md` also carries one later, **Accepted** addendum (2026-07-12, Challenger-reviewed, product-owner sign-off given): a recommended default (**Kiota**, non-binding, per-service-deviable; Refit + Refitter recorded as a valid alternative) for the tool that generates each service's typed HTTP client for the other's ADR-006 endpoint — see `../integration-patterns.md` → "Client-generation tooling." The Architect's original pick was Refit + Refitter; the Challenger flagged that it has no drift-detection mechanism against a stale-regenerated client (unlike Kiota's `kiota-lock.json`), and the product owner chose Kiota for that reason at sign-off. Deliberately kept out of `adr/` as a same-document addendum rather than a seventh ADR, since the recommendation itself concludes it should not be a binding cross-service rule — note this is a judgment call not explicitly carved out by this file's own "What belongs here" criteria above (the Challenger flagged the same tension); revisit if a future non-binding decision makes the ADR-vs-addendum line worth formalizing.
