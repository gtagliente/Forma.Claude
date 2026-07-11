# docs/features/

## Purpose

Holds feature-scoped knowledge: requirements, decisions, and notes for a specific, identifiable piece of product functionality as it's being analyzed or built.

## What belongs here

One subfolder per feature, named `FT-NNN-short-name/` (e.g. `FT-001-exercise-management/`), each containing:

- `README.md` — what the feature is, its current status.
- `requirements.md` — feature-specific requirements.
- `decisions/` — decisions local to this feature only.
- `notes.md` — working notes.

## What does NOT belong here

- Decisions that affect other features or services → promote to `../architecture/adr/` (see Context Promotion Rules in `CLAUDE.md`).
- Domain concepts shared across features → `../product/domain-model.md` or `glossary.md`.

## When an agent should load this context

- Any agent working on a specific, already-identified feature — load only that feature's subfolder, not the whole tree.
- Not relevant during whole-product domain discovery (that's `../product/`).

## Current state

Empty. No features have been scoped yet — iteration 1 is whole-product domain analysis, not feature-level breakdown. Candidate features will likely emerge from the bounded contexts proposed in `../architecture/bounded-contexts.md` in a future iteration.
