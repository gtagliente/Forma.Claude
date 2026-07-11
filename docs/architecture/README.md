# docs/architecture/

## Purpose

Holds the system-wide *technical* structure of Forma: how the domain (defined in `../product/`) is translated into bounded contexts, services (if any), integration patterns, and formally recorded architecture decisions.

## What belongs here

- **System context** — how Forma fits together as a whole and interacts with the outside world.
- **Service map** — the set of services/modules that exist (or are proposed), and their responsibilities. Empty/proposal-only until a service actually exists.
- **Context map** (`context-map.md`) — domain areas, candidate service boundaries, and the relationships between them. Explicitly evaluates alternatives (e.g. modular monolith) rather than assuming microservices.
- **Integration patterns** — how contexts/services are expected to communicate once more than one exists (sync/async, events, etc.).
- **Architecture Decision Records** (`adr/`) — global, cross-cutting decisions only. Per the Context Promotion Rules in `CLAUDE.md`, a decision only lands here once it affects more than one service/feature (security, deployment, cross-service APIs/events, system-wide architecture). Service- or feature-local decisions stay local, in `../services/<service>/decisions/` or `../features/<feature>/decisions/`.

## What does NOT belong here

- Domain/business concepts in business language → `../product/`
- Per-service implementation detail → `../services/<service>/`
- Coding standards / CI / testing process → `../engineering/`

## When an agent should load this context

- **Always** for the Architect agent.
- For the Challenger, to check proposed boundaries against `../product/` for over-engineering or premature complexity.
- For any agent making a decision that might be cross-cutting, to check `adr/` for existing precedent before proposing something new.
