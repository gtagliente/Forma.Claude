# Branch: analysis — Local Decisions

## Current state

Empty by design. Every decision made so far on this branch (user model, Workout versioning/snapshot semantics, offline session logging, Progress Tracking consistency model) turned out to be cross-cutting — affecting more than one bounded context — so each was promoted directly to `../../../architecture/adr/` as it was decided, per the Context Promotion Rules in `CLAUDE.md`, rather than staying here first.

This folder exists for the case where a future decision on this branch is genuinely local (doesn't need to become an ADR) — none have arisen yet.
