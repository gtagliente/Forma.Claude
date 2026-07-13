# Branch: analysis — Pending Items

## Deliberately deferred (not blocking, may return in a future iteration)

- **Body metrics & goals ownership** — Progress Tracking's core (PRs, volume, trends) doesn't need this; revisit only if/when goals become a real feature. See `../../../scratchpad/open-questions/iteration-1.md` (#7).
- **Units & localization** (metric/imperial, language) — #9.
- **Notifications/reminders** for scheduled Routine occurrences — #10.
- **Monetization/business model** — #13.

## Still open, not yet decided (lower stakes, not currently blocking)

From `../../product/requirements-and-open-items.md` → Ambiguities:
- Routine scheduling model (recurring pattern vs. calendar dates, or both).
- Session provenance (must a Session always trace to a Workout, or can it be freestanding/ad hoc?).
- Difficulty attribute (global/objective vs. subjective per user).
- Tags (free-form vs. controlled vocabulary).
- Enrichment trigger mechanism (on-demand vs. automatic).
- Content curator / library maintainer role for the shared Exercise library (`requirements-and-open-items.md` → Users, item 3).

## Next steps for this branch

1. Decide whether to run another analysis iteration on one of the deferred/open items above, or judge the domain/architecture understanding sufficient per `CLAUDE.md`'s bar and move toward implementation scaffolding.
2. ~~Accept an architecture approach~~ **Done** — [ADR-005](../../architecture/adr/ADR-005-microservices-architecture.md): microservices, four services (`identity-service`, `exercise-service`, `training-planning-service`, `training-execution-service`), each with an independent datastore, placeholder folders bootstrapped under `../../services/`.
3. ~~**Inter-service integration pattern**~~ **Done** — [ADR-006](../../architecture/adr/ADR-006-cross-service-reference-integrity.md): governed, direct point-to-point synchronous calls between `exercise-service` and `training-planning-service`, fail-open for the best-effort Exercise-existence check at Workout create/edit, fail-closed for the hard-block Workout-reference check at Exercise delete. No orchestration service, no async eventing adopted at this time. Accepted 2026-07-12. `training-execution-service`'s session-start Workout Version fetch remains separately open, deferred to that service's own future central-loop pass (already resolved *what* it fetches, per ADR-002/ADR-005 — just not *how* yet).
4. Once this branch's goals are met, promote anything still branch-local here into the permanent knowledge base (it should already be minimal, since decisions have been promoted to ADRs as they were made) and consider this branch's context closed per `../README.md`.
