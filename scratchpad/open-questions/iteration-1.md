# Open Questions — Iteration 1

Unresolved items surfaced by the Analyst/Architect/Challenger pass, requiring future iterations or human input. Prioritized roughly by how much downstream work depends on the answer.

## High priority (blocks meaningful architecture progress)

1. **Who is the user?** Solo athlete only, or does Forma support a coach directing another person's training? (`../../docs/product/requirements-and-open-items.md` → Users) This determines whether an Identity/delegation context exists at all, and affects data ownership across every other area.
2. **Is a Workout Session tied to a snapshot of the plan, or a live reference?** I.e. if a Workout is edited, do in-progress or historical Sessions reflect the change? (`../../docs/architecture/bounded-contexts.md` → Context 3, flagged by Challenger as a product decision, not an architecture default.)
3. **Routine↔Workout reference semantics**: does editing a Workout retroactively change what a Routine "means" for schedule occurrences already passed or in progress? Related to #2.

## Medium priority (shapes the data/domain model)

4. **Is the Exercise library shared/curated, or can individual users define private exercises (or both)?**
5. **Is "Set" a first-class, addressable concept**, or an inline attribute list on Workout/Workout Session? Needed before Training Execution's storage shape is designed.
6. **Exercise variation/parameterization**: is "Barbell Bench Press" vs. "Dumbbell Bench Press" one Exercise with parameters, or two separate Exercises?
7. **Who owns body metrics and goals** (bodyweight, measurements, targets)? Not currently assigned to any domain area, but plausibly relevant to Progress Tracking.
8. **AI enrichment trust model**: auto-merged into what a user sees, or held as a separate, clearly-labeled suggestion pending acceptance?

## Lower priority (can wait for later iterations)

9. Units & localization (metric/imperial, language).
10. Notifications/reminders for scheduled Routine occurrences.
11. Offline/connectivity handling during a training session.
12. Media resource handling for Exercises (capture, storage, size limits).
13. Monetization/business model (out of domain-modeling scope, but worth knowing eventually).
14. Historical mutability: can a completed Workout Session be edited/deleted, and if so, what happens to Progress Tracking computations already derived from it?

## Recommended next analysis iteration

Focus iteration 2 on **question 1 (user/coach model)** and **question 2 (snapshot vs. live reference)** specifically — both are flagged by the Challenger as high-leverage and currently being implicitly defaulted rather than decided. Once resolved, re-run the Architect pass on `bounded-contexts.md` and `context-map.md`, since both may change shape (especially whether an Identity context needs to exist, and whether Training Execution's snapshot proposal should become an ADR).
