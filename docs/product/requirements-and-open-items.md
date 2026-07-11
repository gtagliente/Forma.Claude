# Forma — Requirements, Users, and Open Items (Iteration 1)

_Analyst output. This is a discovery document, not a spec — many items below are questions, not decisions._

## Users (decided — [ADR-001](../architecture/adr/ADR-001-user-model-iteration-1.md))

`CLAUDE.md` never explicitly names a user/persona; the product owner has now resolved this for the current iteration:

1. **Solo athlete / normal user** — manages their own exercises/workouts/routines and logs their own sessions. Confirmed as the only persona for this iteration.
2. **Coach/trainer directing another person** — explicitly **not modeled** this iteration (see [ADR-001](../architecture/adr/ADR-001-user-model-iteration-1.md) and glossary "Explicitly out of scope"). May be revisited later if a real requirement emerges.
3. **Content curator / library maintainer** — still **open**. The shared Exercise library's governance (who can add to it, beyond a user promoting their own private Exercise — see domain-model.md → Exercise) is not yet specified.

## Main workflows (as inferable from the described lifecycle)

1. **Define an exercise** — create/maintain an Exercise (name, instructions, equipment, media, tags, difficulty); optionally trigger/receive AI enrichment.
2. **Compose a workout** — select exercises, define sets/reps/weight/duration/rest/sequencing/supersets/circuits, save as a reusable Workout.
3. **Build a routine** — arrange Workouts across a schedule (e.g. weekly).
4. **Execute a session** — start a Workout Session (presumably from a Routine's scheduled occurrence, or a Workout directly, or possibly ad hoc), record actual reps/weight/duration per set as training happens.
5. **Review progress** — view trends, PRs, volume, consistency derived from session history.

Workflow 4 in particular is under-specified — see open items.

## Missing requirements (not addressed at all in `CLAUDE.md`)

- **Identity & access** — accounts, authentication, authorization, data ownership/privacy. `CLAUDE.md` says nothing; the persona question is now resolved ([ADR-001](../architecture/adr/ADR-001-user-model-iteration-1.md) — single normal user), but auth/account mechanics themselves are still undesigned.
- **Multi-user relationships** — sharing, following, coach-athlete, social features. `CLAUDE.md` says nothing, and [ADR-001](../architecture/adr/ADR-001-user-model-iteration-1.md) explicitly excludes coach-athlete delegation from this iteration; general social/sharing features remain unaddressed.
- **Units & localization** — kg vs lb, metric vs imperial, language. **Explicitly deferred** — a deliberate scope choice for this iteration, not an oversight.
- **Body metrics / goals** — bodyweight, measurements, target goals. **Explicitly deferred** — Progress Tracking's core (PRs, volume, trends) is fully derivable from Workout Session data alone, so this was never a blocker; see `../../scratchpad/open-questions/iteration-1.md` (#7).
- **Notifications/reminders** — nothing said about reminding a user of a scheduled Routine day. **Explicitly deferred** — a deliberate scope choice for this iteration, not an oversight.
- ~~**Offline / connectivity**~~ — **Resolved**: Workout Session logging (only) works offline, caching locally and syncing on reconnect. See [ADR-003](../architecture/adr/ADR-003-offline-workout-session-logging.md).
- ~~**Media handling**~~ — **Resolved**: a Media Resource concept (uploaded file or external link) attaches to Exercise and Workout Session. Capture/storage mechanics remain implementation detail. See `domain-model.md` → Media Resource.
- **Monetization / business model** — entirely absent; may not matter for domain modeling but affects scope. **Explicitly deferred** — out of scope for domain modeling.
- ~~**Exercise library governance**~~ — **Resolved**: both a shared/curated library and private per-user Exercises exist; private Exercises are visible only to their creator, with a promotion-to-shared mechanism deferred to a future iteration. See `domain-model.md` → Exercise. Curatorship of the *shared* library itself (item 3 under Users, above) remains open.

## Ambiguities in what IS described

- ~~**Workout ↔ Routine reference semantics**~~ — **Resolved**: Routine tracks the latest Workout Version (live); Workout Session pins the version current when it started. See [ADR-002](../architecture/adr/ADR-002-workout-versioning-and-session-snapshot.md).
- **Routine scheduling model**: recurring weekly pattern (as shown in the example) vs. specific calendar dates — or both? Still open.
- **Session provenance**: must every Workout Session trace back to a specific Workout (and via a Routine occurrence), or can sessions be freestanding ("did an ad hoc session today")? Still open.
- ~~**Set-level granularity**~~ — **Resolved**: Set is an inline ordered entry embedded in its parent Workout Version / Workout Session, not an independently addressable concept. See `domain-model.md` → Set.
- **Exercise variation/parameterization** — **Resolved**: modeled via an Exercise parent/child (generalization/specialization) hierarchy, not as one Exercise with an equipment parameter. See `domain-model.md` → Exercise.
- **Difficulty attribute**: global/objective (set once per Exercise) or subjective per user? Still open.
- **Tags**: free-form or controlled vocabulary — matters for search/filtering and for AI enrichment consistency. Still open.
- ~~**AI enrichment trigger & trust**~~ — **Resolved**: enrichment stays separate and clearly labeled, never auto-merged; a promotion mechanism lets a user explicitly accept a suggestion into the Exercise's standard data. See `domain-model.md` → Enrichment. Trigger mechanism (on-demand vs. automatic) is still open.

## Recommendation for next iteration

Every originally-flagged open item is now either resolved (ADR-001 through ADR-004, plus the domain-model additions for Exercise ownership/hierarchy, Set, Enrichment promotion, and Media Resource) or explicitly deferred as a deliberate scope choice (body metrics/goals, units/localization, notifications, monetization — see `../../scratchpad/open-questions/iteration-1.md`). There is no outstanding blocking question from Iteration 1; the next iteration is free to revisit a deferred item or move toward implementation scaffolding.
