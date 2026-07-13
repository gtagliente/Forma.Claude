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

## Cross-context reference integrity (Iteration 2)

### Trigger

`training-planning-service` and `exercise-service` have both started real implementation against ID-only cross-service references (ADR-005's no-shared-database rule: no cross-service joins/foreign keys, references resolved via API call or denormalized copy). Two concrete situations surfaced the same general question — what should happen, from the user's point of view, when a reference crosses a bounded-context boundary, both at the moment the reference is created and if the referenced thing is later deleted:

1. **Workout creation → Exercise existence.** `training-planning-service` just built Workout creation, referencing `Exercise` by `ExerciseId` only, currently unvalidated (`Forma.Resource/Forma.Planner`).
2. **Exercise deletion → Workout reference.** `exercise-service` just built Exercise deletion and flagged, without solving, that nothing today can prevent or detect deleting an Exercise a Workout references (`Forma.Exercise/docs/features/FT-003-update-delete.md` → Central Architect Gate).

This is analyzed here purely as business rule / user experience — *how* the system enforces or checks any of this (synchronous call, async event, projection, etc.) is explicitly left to the Architect; see open questions below.

### Business rules resolved

**1. Workout creation/editing referencing an Exercise — best-effort validity, not a hard transactional guarantee.**

In normal use, a user composes a Workout by picking Exercises from a list they can already see (their own private Exercises plus the shared library) — by construction, the overwhelming majority of references are valid the moment they're made. The residual risk is a narrow race window: the Exercise is deleted between the user browsing/loading it and the user saving the Workout. That window is narrow, low-frequency, and — because the user is actively inside the Workout-editing flow when it would surface — cheap and immediate to recover from (just re-pick). A strict, unconditional "the referenced Exercise must exist or the save is rejected" guarantee is therefore **not** a genuine product requirement. The system *should* make a reasonable effort to catch an invalid reference at save time (catching the common case early, with a clear message), but that check is a UX safeguard, not something the user's ability to save should hard-depend on.

What **is** a hard requirement, regardless of how good that check turns out to be: a Workout (or Workout Version) that ends up holding a **Dangling Reference** (see glossary) must degrade gracefully wherever it is later shown or acted on — the affected entry presented clearly as unavailable/removed, never silently dropped, never a generic error, never corrupted display data. This applies to viewing/editing the Workout itself, to a Routine that references it, and to attempting to start a Training Session from it (see open question 2, below).

**2. Exercise deletion while referenced by a Workout — blocked by default, same rule for shared and private.**

Deleting an Exercise that is still referenced by at least one Workout should be **blocked**, surfaced as a clear, actionable message. This directly extends a pattern already shipped and accepted: `exercise-service`'s existing "cannot delete an Exercise that has children in the hierarchy" rule (FT-003) — this is the same shape of guard (block deletion while something still depends on this Exercise), just crossing a service boundary instead of staying within one. The user must first remove the Exercise from the Workout(s) referencing it (via a new Workout Version, per the existing versioning model) before the delete is allowed.

The rule applies identically to a private Exercise (referenced only by its own owner's Workouts — a private Exercise is visible only to its creator, so nobody else could have referenced it) and a shared/library Exercise (potentially referenced by many other users' Workouts). A single, predictable delete-blocking behavior regardless of ownership is preferable to one that behaves differently depending on it — a broken reference damages a Workout's integrity the same way no matter who owns the Exercise that broke it.

That said, the ownership model does surface a real, worth-flagging asymmetry (see open question 3): a user deleting their **own private** Exercise always has full agency to resolve the block themselves (they own every Workout that could reference it). A user attempting to delete a **shared** Exercise does not — the referencing Workouts may belong to other users entirely outside their control, meaning "block until unreferenced" could make a widely-adopted shared Exercise practically undeletable. Flagged, not resolved.

**Rejected as the default behavior**: allowing the delete to proceed and leaving affected Workouts with a silently broken reference, relying solely on rule 1's graceful-degradation fallback. Graceful degradation is the right *safety net* for the narrow, unavoidable race-condition case in rule 1 — but making broken references a routine, expected outcome of ordinary Exercise deletion, rather than a rare edge case, would undermine the core value of the Workout concept (a plan the user can trust, not one that silently erodes — see `vision.md`). Blocking is also the cheaper path: it needs no new domain concept (no soft-delete/archival) and doesn't reopen `exercise-service`'s already-shipped hard-delete-only decision.

**Deliberately deferred (avoid premature complexity)**: an explicit override ("delete anyway, understanding it affects N workouts") is a plausible future refinement — especially for private Exercises, where the deleting user bears the full consequence themselves — but isn't required this iteration, and isn't recommended for shared Exercises at all (the deleting user there isn't the one bearing the consequence, so an override would bypass other users' implicit expectation that their own plans stay intact). Likewise, a softer "archive/retire from the library, but stay resolvable for existing references" treatment specifically for shared Exercises is a plausible longer-term answer to the "undeletable once popular" tension above — flagged, not adopted, since it's a new domain concept entangled with the still-open shared-library governance question (see Users, item 3) and isn't justified by a demonstrated need yet.

### Is this a one-off (Exercise↔Workout), or a general shape?

**A general shape, not a one-off.** ADR-005's independent-datastore rule means *every* arrow crossing a service boundary in the context map is a candidate for this same pair of questions (creation-time validity + deletion-time protection). Instances already identified or clearly imminent, beyond Exercise↔Workout:

- **User ↔ everything.** Every owning concept in every service (Exercise, Workout, Routine, Workout Session) carries an `OwnerId`/User reference into `identity-service`. Account deletion is a normal, expected lifecycle event — unlike deleting a shared-library Exercise, which is comparatively rare — making this arguably the *most* certain next occurrence of this shape. Worth revisiting explicitly once `identity-service` moves past its current placeholder.
- **Workout ↔ Routine deletion.** The same-service (intra-`training-planning-service`) twin of rule 2 above. That service's own backlog already lists Workout/Routine Update-Delete as not yet built (`Forma.Resource/Forma.Planner`). When Workout delete is built, it will need the identical "block while a Routine still references it" rule — already covered by the same precedent, flagged here only so it isn't re-derived from scratch when that feature comes up.
- **Routine ↔ Workout Session**, contingent on how the still-open "session provenance" question resolves (does a session always trace to a Workout via a Routine, or can a Routine reference an occurrence directly?) — if it resolves toward a direct reference, that is a third Training-Planning ↔ Training-Execution instance, alongside the already-decided Workout-Version-pinning one (ADR-002/ADR-005).

Given one near-certain (User) and one imminent (Workout↔Routine) additional occurrence beyond the two resolved this iteration, the recommendation to the Architect is to treat this as a **general policy** — a repeatable rule for how a referencing context behaves at creation and how a referenced context behaves at deletion, whenever a cross-service ID reference exists — rather than re-deriving a bespoke answer per concept pair. The one already-settled carve-out to carry forward: a **denormalized/pinned copy is exempt from both concerns by design** — a Workout Session's pinned Workout Version is deliberately immune to the referenced Workout's later deletion or edits (ADR-002/ADR-005), and that's correct, not a gap this policy needs to also cover.

### Open questions for the Architect

1. **Mechanism for both checks** — the creation-time best-effort check (rule 1) and the deletion-time blocking check (rule 2) both require `exercise-service` and `training-planning-service` to answer "does X exist / is X referenced" across the service boundary; it's the same underlying capability in both directions. Whether it's synchronous, async/eventually-consistent, or something else is the Architect's call (`../architecture/integration-patterns.md`, still empty). One business-level input to weigh: for rule 2, a **false negative** (check says "not referenced" when it actually is) lets a delete slip through and orphans a reference — the exact outcome the rule exists to prevent, so the higher-stakes failure mode. A **false positive** (says "referenced" when it's not) merely blocks a delete that should have succeeded — an inconvenience, not a correctness problem. This asymmetry may help decide how fresh/authoritative the check needs to be.
2. **Session-start behavior when a Workout's Exercise reference no longer resolves.** Rule 1 establishes that a Dangling Reference must be shown gracefully rather than error — but what should happen specifically when a user tries to start a Training Session from a Workout Version that has one or more unresolved Exercise references? Product intent (not yet a full requirement): starting a session shouldn't hard-fail solely because one entry is broken — the user should be able to proceed with the still-valid entries, with a clear indicator on the broken one — but what the denormalized session snapshot actually captures/shows for that entry is bound up with the pinning/denormalization mechanism (ADR-002/ADR-005) and is the Architect's to resolve.
3. **Shared-Exercise "undeletable once popular" tension** (flagged above, not resolved) — worth a joint look once the shared-library governance/curatorship question (Users, item 3) is addressed, since any softer treatment for shared Exercises (e.g. archive/retire) is entangled with who's even allowed to delete one in the first place.
4. **User/account deletion across all four services** — not solved here, flagged as the next, near-certain occurrence of this same shape once `identity-service` is real. No immediate action needed; just don't let it surface as a surprise later.

## Recommendation for next iteration

_Iteration 1_: every originally-flagged open item is now either resolved (ADR-001 through ADR-004, plus the domain-model additions for Exercise ownership/hierarchy, Set, Enrichment promotion, and Media Resource) or explicitly deferred as a deliberate scope choice (body metrics/goals, units/localization, notifications, monetization — see `../../scratchpad/open-questions/iteration-1.md`).

_Iteration 2_: the cross-context reference integrity question (above) is resolved at the business-rule level for its two triggering instances (Exercise↔Workout, both directions). Four items are now handed to the Architect (see "Open questions for the Architect," above) — none block further product analysis; they block the concrete implementation of Exercise deletion's safeguard and Workout creation's validation, which is Architect/service-loop scope from here.
