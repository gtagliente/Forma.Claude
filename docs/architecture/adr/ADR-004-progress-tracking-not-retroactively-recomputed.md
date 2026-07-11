# ADR-004: Progress Tracking Computations Are Not Retroactively Recomputed

## Status

Accepted.

## Context

`requirements-and-open-items.md` and the Challenger review both flagged historical mutability as unaddressed: can a completed Workout Session be edited or deleted after the fact, and if so, what happens to Progress Tracking values (PRs, training volume, trends) already derived from it? See `../../../scratchpad/open-questions/iteration-1.md` (#14).

This decision fixes the consistency model between Training Execution and Progress Analytics, so it is cross-cutting per the Context Promotion Rules.

## Decision

A completed Workout Session **can** be edited or deleted after the fact. However, Progress Tracking values already computed from it (e.g. a Personal Record, a period's training volume, a trend) are **not retroactively recomputed** when the underlying session changes. Those values stay exactly as they were at the moment they were computed. Only *future* Progress Tracking computations reflect the edited/deleted session.

## Alternatives considered

- **Full recompute on every edit**: rejected. Recomputing all downstream analytics (PRs, volume, trend series) every time a historical session changes is a real technical cost, and more importantly a product-consequence risk — it means a user's past achievements (e.g. "you hit a new PR that day") could silently disappear or change after the fact because of an unrelated data correction. This conflicts with the historical-immutability philosophy already established for Workout↔Session in ADR-002: history, once recorded and observed, should not silently rewrite itself.

## Consequences

- **Progress Analytics** (currently a capability inside Training Execution per `context-map.md`) computes values once and treats them as durable facts, not live-recalculated views.
- Editing/deleting a session is still a legitimate, supported user action (this ADR does not make sessions immutable) — it just doesn't cascade backward into already-surfaced analytics.
- If a future iteration decides analytics must always reflect current session data (e.g. because "stale PRs" turns out to be a real user complaint), that would require revisiting this ADR explicitly, not silently defaulting to recompute-on-write.

## References

- `../../product/domain-model.md` → Workout Session, Progress Tracking
- `ADR-002-workout-versioning-and-session-snapshot.md`
- `../../../scratchpad/challenger-review-iteration-1.md`
- `../../../scratchpad/open-questions/iteration-1.md` (#14)
