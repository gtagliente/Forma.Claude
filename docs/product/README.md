# docs/product/

## Purpose

Holds the shared understanding of Forma as a *product*: who it's for, what it does, and the business/domain concepts it's built from — independent of any technical implementation.

## What belongs here

- **Vision** — why the product exists, who it serves, what "done" looks like directionally (not a roadmap).
- **Requirements** — functional needs, user workflows, constraints, open/missing requirements.
- **Domain model** — the core business concepts (Exercise, Workout, Routine, Workout Session, Progress Tracking, and whatever else discovery surfaces), their attributes, and their relationships — described in business language, not data-schema language.
- **Glossary** — canonical definitions of domain terms, so every agent and every doc uses the same words to mean the same things.

## What does NOT belong here

- Technical architecture, bounded contexts, service boundaries → `../architecture/`
- API contracts, persistence schemas, feature-specific implementation detail → each service's own repo (e.g. `Forma.Exercise`, `Forma.Resource`), not mirrored here

## When an agent should load this context

- **Always** for the Analyst agent — this is its primary output and input for every iteration.
- For the Architect agent, as the *input* to propose bounded contexts (never invents domain concepts of its own).
- For any agent about to touch a feature, to check the domain model/glossary before introducing new terminology.
- For the Challenger, to verify architecture/feature decisions haven't drifted from the agreed domain model.
