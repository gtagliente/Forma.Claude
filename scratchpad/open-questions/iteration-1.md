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
7. **Who owns body metrics and goals** (bodyweight, measurements, targets)? Not currently assigned to any domain area, but plausibly relevant to Progress Tracking. **Still open — now the highest-priority remaining item.**
8. ~~**AI enrichment trust model**~~ **Resolved** — kept separate/labeled, never auto-merged; a promotion mechanism lets a user accept a suggestion into the Exercise's standard data. See `../../docs/product/domain-model.md` → Enrichment.

## Lower priority (can wait for later iterations)

9. Units & localization (metric/imperial, language).
10. Notifications/reminders for scheduled Routine occurrences.
11. Offline/connectivity handling during a training session.
12. Media resource handling for Exercises (capture, storage, size limits).
13. Monetization/business model (out of domain-modeling scope, but worth knowing eventually).
14. Historical mutability: can a completed Workout Session be edited/deleted, and if so, what happens to Progress Tracking computations already derived from it?

## Recommended next analysis iteration

Questions 1–6 and 8 are resolved (see [ADR-001](../../docs/architecture/adr/ADR-001-user-model-iteration-1.md), [ADR-002](../../docs/architecture/adr/ADR-002-workout-versioning-and-session-snapshot.md), and the updated `domain-model.md`/`bounded-contexts.md`/`context-map.md`). Focus iteration 2 on **question 7 (body metrics/goals ownership)** — the highest-leverage remaining gap, since it's plausibly relevant to Progress Tracking but currently unowned by any domain area. The "lower priority" items (9–14) remain fair game for whenever they become blocking.
