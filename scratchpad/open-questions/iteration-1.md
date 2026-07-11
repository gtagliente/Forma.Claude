# Open Questions — Iteration 1

Unresolved items surfaced by the Analyst/Architect/Challenger pass, requiring future iterations or human input. Prioritized roughly by how much downstream work depends on the answer.

## High priority (blocks meaningful architecture progress)

1. ~~**Who is the user?**~~ **Resolved** — single normal-user persona, no coach/delegation. See [ADR-001](../../docs/architecture/adr/ADR-001-user-model-iteration-1.md).
2. ~~**Is a Workout Session tied to a snapshot of the plan, or a live reference?**~~ **Resolved** — Workout is versioned; a Session pins the version current when it started. See [ADR-002](../../docs/architecture/adr/ADR-002-workout-versioning-and-session-snapshot.md).
3. ~~**Routine↔Workout reference semantics**~~ **Resolved** — Routine tracks the latest Workout version live. See [ADR-002](../../docs/architecture/adr/ADR-002-workout-versioning-and-session-snapshot.md).

## Medium priority (shapes the data/domain model)

4. ~~**Is the Exercise library shared/curated, or can individual users define private exercises (or both)?**~~ **Resolved** — both; private exercises are visible only to their creator, promotion-to-shared deferred. See `../../docs/product/domain-model.md` → Exercise.
5. ~~**Is "Set" a first-class, addressable concept**, or an inline attribute list on Workout/Workout Session?~~ **Resolved** — inline ordered entries, no independent identity. See `../../docs/product/domain-model.md` → Set.
6. ~~**Exercise variation/parameterization**~~ **Resolved** — modeled via a parent/child (generalization/specialization) hierarchy between Exercises. See `../../docs/product/domain-model.md` → Exercise.
7. **Who owns body metrics and goals** (bodyweight, measurements, targets)? **Explicitly deferred** — a deliberate scope choice. Progress Tracking's core (PRs, volume, trends) is already fully derivable from Workout Session data alone; body metrics/goals were only ever a *possible* future input (`../../docs/product/domain-model.md` → Progress Tracking), not a blocker to the current lifecycle. Revisit in a later iteration — when it returns, note that any body-metric-linked Goal will need a home outside Identity, which stays deliberately minimal per [ADR-001](../../docs/architecture/adr/ADR-001-user-model-iteration-1.md).
8. ~~**AI enrichment trust model**~~ **Resolved** — kept separate/labeled, never auto-merged; a promotion mechanism lets a user accept a suggestion into the Exercise's standard data. See `../../docs/product/domain-model.md` → Enrichment.

## Lower priority (can wait for later iterations)

9. **Units & localization** (metric/imperial, language). **Explicitly deferred** — a deliberate scope choice, not modeled this iteration.
10. **Notifications/reminders** for scheduled Routine occurrences. **Explicitly deferred** — a deliberate scope choice, not modeled this iteration.
11. ~~**Offline/connectivity handling during a training session.**~~ **Resolved** — Workout Session logging (only) works offline, caching locally and syncing on reconnect. See [ADR-003](../../docs/architecture/adr/ADR-003-offline-workout-session-logging.md).
12. ~~**Media resource handling for Exercises**~~ **Resolved** — new Media Resource concept (uploaded file or external link), attachable to Exercise *and* Workout Session. Capture/storage mechanics (size limits, hosting) remain implementation detail. See `../../docs/product/domain-model.md` → Media Resource.
13. **Monetization/business model**. **Explicitly deferred** — out of domain-modeling scope for now.
14. ~~**Historical mutability**~~ **Resolved** — a completed Workout Session can be edited/deleted, but Progress Tracking computations already derived from it are **not** retroactively recomputed; only future computations reflect the change. See [ADR-004](../../docs/architecture/adr/ADR-004-progress-tracking-not-retroactively-recomputed.md).

## Recommended next analysis iteration

Every item in this file is now either resolved (ADR-001 through [ADR-004](../../docs/architecture/adr/ADR-004-progress-tracking-not-retroactively-recomputed.md), plus the corresponding `domain-model.md`/`bounded-contexts.md`/`context-map.md` updates) or explicitly and deliberately deferred (#7, #9, #10, #13). There is no outstanding blocking question from Iteration 1 — the next iteration is free to start fresh, whether that's revisiting a deferred item (body metrics/goals is the most likely candidate to return) or moving toward implementation scaffolding.
