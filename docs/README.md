# Forma — Project Intelligence

This directory is the persistent, file-based memory of the Forma project. Per `CLAUDE.md`, conversation context is temporary — anything worth remembering across sessions and agents must be written here.

## Structure

| Folder | Scope | Contains |
|---|---|---|
| [product/](product/README.md) | Whole product | Vision, requirements, domain model, glossary |
| [architecture/](architecture/README.md) | Whole system | System context, service map, context map, ADRs |
| [engineering/](engineering/README.md) | Whole system | Coding standards, git workflow, testing, devops |
| [../.claude/agents/](../.claude/agents/) | Whole system | Role definitions for each AI agent used on the project — live Claude Code subagents (`analyst`, `architect`, `challenger`), not markdown docs |
| [features/](features/README.md) | Single feature | Per-feature requirements, decisions, notes |
| [branches/](branches/README.md) | Single git branch | Branch-scoped, temporary working context |
| [services/](services/README.md) | Single service | Per-service domain, architecture, API contracts, decisions |

`../scratchpad/` (outside `docs/`) holds ephemeral, unreviewed material — Challenger findings and open questions — that has not yet been promoted into the durable structure above.

## How knowledge should move

Per `CLAUDE.md`'s Context Promotion Rules: knowledge starts as local as possible (a feature note, a service decision, a scratchpad entry) and is only promoted upward — to `architecture/adr/`, `product/`, etc. — when it becomes relevant outside its original scope. Don't pre-promote speculative decisions; don't leave cross-cutting decisions stranded in a local scope either.

## Current state (as of iteration 1)

No services, features, or branches have been defined yet — those folders currently contain only their explanatory `README.md`. The project is still in the domain-discovery phase; see `product/` and `architecture/` for the first analysis iteration, and `../scratchpad/open-questions/` for what's still unresolved.
