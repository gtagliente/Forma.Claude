# docs/engineering/

## Purpose

Holds the practical, cross-cutting engineering conventions used once implementation begins: how code is written, how branches/PRs flow, how testing works, how things get deployed.

## What belongs here

- **Coding standards** — style, patterns, naming, language/framework conventions once chosen.
- **Git workflow** — branching model, commit/PR conventions.
- **Testing strategy** — what gets tested, at what level, with what tools.
- **DevOps** — build, CI/CD, environments, deployment process.

## What does NOT belong here

- Product/domain knowledge → `../product/`
- Architecture decisions (why a boundary exists) → `../architecture/`
- Anything specific to one service's internal implementation → `../services/<service>/`

## When an agent should load this context

- Any implementation/coding agent, before writing or modifying code.
- Not relevant to Analyst/Architect/Challenger work during domain discovery.

## Current state

Empty. No technology choices or engineering conventions have been made yet — this project is still in the domain/architecture discovery phase (see `../product/` and `../architecture/`). This folder will be populated once implementation begins.
