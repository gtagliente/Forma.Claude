# Forma — Project Intelligence

This directory is the persistent, file-based memory of the Forma project. Per `CLAUDE.md`, conversation context is temporary — anything worth remembering across sessions and agents must be written here.

## Structure

| Folder | Scope | Contains |
|---|---|---|
| [product/](product/README.md) | Whole product | Vision, requirements, domain model, glossary |
| [architecture/](architecture/README.md) | Whole system | Architecture options, bounded contexts, context map, ADRs |
| [engineering/](engineering/README.md) | Whole system | Coding standards, git workflow, testing, devops |
| [../.claude/agents/](../.claude/agents/) | Whole system | Role definitions for each AI agent used on the project — live Claude Code subagents (`analyst`, `architect`, `challenger`), not markdown docs |

This repo is deliberately flat and holds only knowledge that's global to the whole
product/system. Per-service, per-feature, and per-branch implementation detail lives
in that service's own repo (e.g. `Forma.Exercise`, `Forma.Resource`) — it isn't
mirrored here.

`../scratchpad/` (outside `docs/`) holds ephemeral, unreviewed material — Challenger findings and open questions — that hasn't fed back into `product/`/`architecture/` yet.

## How knowledge should move

A decision that's local and reversible (scoped to one service or feature) doesn't need a doc in this repo at all — the code, commit message, and that service's own PR description are enough. Only decisions that are costly to reverse or cross a service/API/security/deployment boundary get written up, as a short ADR in `architecture/adr/`. See that folder's `README.md` for the exact bar.

## Current state (as of iteration 1)

See `product/` and `architecture/` for the first analysis iteration, and `../scratchpad/open-questions/` for what's still unresolved.
