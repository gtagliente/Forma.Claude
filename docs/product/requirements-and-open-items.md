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

Workflow 4's live in-session mechanics (pacing timers, rest, progress, completion) are now specified — see "Live Workout Session Execution (Iteration 4)," below. Provenance (whether every session must trace to a Workout/Routine, or can be ad hoc) remains open — see Ambiguities, below.

## Missing requirements (not addressed at all in `CLAUDE.md`)

- **Identity & access** — accounts, authentication, authorization, data ownership/privacy. `CLAUDE.md` says nothing; the persona question is resolved ([ADR-001](../architecture/adr/ADR-001-user-model-iteration-1.md) — single normal user). Authentication mechanics themselves, undesigned as of Iteration 1/2, are now actively being closed: `identity-service` already issues real bearer tokens, and `exercise-service`/`training-planning-service` are being changed to derive the acting user from a validated token instead of trusting a caller-supplied id. See "Authenticated request identity," below, for the resolved business requirement and what's handed to the Architect. Authorization/roles beyond the single-persona model (e.g. a content-curator role for the shared library) remain explicitly out of scope — see Users, item 3.
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

## Authenticated request identity (Iteration 3)

### Trigger

`exercise-service` and `training-planning-service` were deliberately built with a caller-supplied `RequestingUserId`/`OwnerId` parameter standing in for real authentication (`Forma.Exercise/docs/features/FT-001-ownership-visibility.md` → "Auth stand-in"), explicitly flagged at the time as a gap to close once `identity-service` could provide real auth. `identity-service` (`Forma.Resource/Forma.Auth`) already issues real bearer tokens today (JWT) — it was never a placeholder for the authentication *mechanism* itself, only for the other two services' consumption of it. That consumption gap is now being closed, starting with `exercise-service`, then `training-planning-service` (near-identical architectures).

### Business rules resolved

**1. "A request is authenticated" means every operation acts strictly on behalf of one verified identity — this enforces an already-decided concept, it does not introduce a new one.**

Given the single normal-user model ([ADR-001](../architecture/adr/ADR-001-user-model-iteration-1.md)), closing this gap does not add anything to the domain model: ownership (a user owns their own Exercises/Workouts/Routines — `domain-model.md` → Exercise, FT-001) was already decided; what was missing was enforcement. The requirement: a user must never be able to view, list, edit, or delete another user's private data by supplying a different id, and any operation scoped to "my data" must act on the identity the token proves, never on an id the caller merely asserts. This was already FT-001's implicit, flagged expectation — this work closes that gap. It does not reopen ADR-001, and it does not introduce roles, permissions, or any Identity concept beyond the single `User` already confirmed minimal in `bounded-contexts.md` → Identity.

**2. Shared-library reads plausibly shouldn't require a token — flagged as a product decision, not defaulted silently.**

The visibility model (`OwnerId == null` = shared, visible to everyone — FT-001) implies a natural split: any operation that could expose or act on a specific user's *private* data must require a verified identity; a read that only ever returns shared-library content exposes no private data, and nothing in `CLAUDE.md` or prior analysis suggests a user should need an account just to browse the shared Exercise library. Recommendation to the Architect: require authentication for anything touching a user's own data (their private Exercises/Workouts/Routines, any create/edit/delete, anything filtered to "mine"); do not require it for a pure shared-library browse. This is a recommendation, not a locked decision — flagged explicitly so the product owner can confirm or override rather than have it decided implicitly by whichever way the Architect finds easiest to implement.

**3. The "content curator" open item (Users, item 3) is unaffected and stays out of scope.**

Deciding *who* is allowed to create a shared/unowned Exercise is a materially bigger decision (a real authorization/roles model) than "verify who the caller claims to be." Folding it into this work would be exactly the premature complexity `CLAUDE.md` warns against, with no concrete requirement forcing it now. This work verifies identity; it does not add authorization tiers. The curator question remains open, untouched by this iteration.

### Open questions for the Architect

1. **Token validation and failure handling** — how each service validates the token (signature, expiry) and what a request with a missing/invalid/expired token gets back, is a technical decision, not a business rule; the business requirement is only that no operation touching a user's private data may proceed without a validated identity behind it.
2. **Whether to require authentication universally, or leave shared-library reads open** (business rule 2, above) — recommend confirming explicitly with the product owner rather than defaulting either way based on implementation convenience.

## Read-after-write freshness (Iteration 3)

### Trigger

Product owner report: after creating or editing an Exercise or a Workout in the frontend, the change doesn't appear to show up. Confirmed root cause, taken here as ground truth (not re-derived — this is a technical defect, not a domain question): `exercise-service` and `training-planning-service` cache list-query results under a per-user-scoped cache key, but their event handlers invalidate the cache using an unscoped key, so invalidation never matches the real cached entry — users see stale lists until the cache's own TTL expires. Identical shape across Exercise/Workout/Routine in both services.

### Business rule resolved

**A user must see their own create/edit/delete reflected on their very next list or view of that data — this is a correctness guarantee, not an eventual-consistency window a user should ever have to tolerate.**

There is no legitimate product reason for a user's own edit to be invisible to themselves afterward. Framed as a requirement: after any operation that creates, edits, or deletes a user's own Exercise, Workout, or Routine, that same user's subsequent list/view requests for that data must reflect the change — immediately, not "eventually, once a cache expires." This is a defect against an already-intended behavior, not a new domain concept: the per-user cache key was deliberately introduced by FT-001 specifically to scope visibility correctly (`Forma.Exercise/docs/features/FT-001-ownership-visibility.md` → "Read side"); only the invalidation side wasn't kept in sync. No `domain-model.md` change follows from this — it's a clear requirement statement for the Architect/service-loop fix to be checked against.

**Scope: read-after-your-own-write is the hard requirement; instant cross-user propagation on shared content is not.**

For a *shared* Exercise, one user's edit becoming visible to *other* users within the existing cache window (currently 2h absolute / 60s sliding) is acceptable — nothing in prior analysis establishes an expectation that one user's edit propagate instantly to every other viewer of shared content, the way there is for the editor's own next read of their own write. Recommendation to the Architect: the fix must guarantee the acting user's own subsequent reads are fresh; guaranteeing instant freshness for every other user's view of a shared-library change is not a requirement this iteration, even if a correct fix happens to deliver it as a side effect.

### Open questions for the Architect

None — this is a defect against an already-decided requirement (FT-001's per-user visibility scoping), not a new open item. The fix itself (aligning the cache key used for invalidation with the one used for storage) is Architect/service-loop scope.

## Live Workout Session Execution (Iteration 4)

### Trigger

Product owner request: build the actual "start a workout session" real-time execution experience — a play button on a Workout's detail page starts a session at its first exercise; an up-counting active timer runs while the user performs an exercise; a down-counting rest timer (with a "+seconds" control) runs between exercises; the flow auto-advances to the next exercise when rest ends; this repeats until all exercises are done; an overall progress bar shows completion. This refines Workout Session's existing shape (`domain-model.md`) rather than introducing a new concept, and must respect three already-accepted constraints: ADR-002 (session pins the Workout Version at start), ADR-003 (session logging is offline-capable, set-by-set), and ADR-004 (Progress Tracking values aren't retroactively recomputed).

### Business rules resolved

**1. The live flow steps through sets, not just exercises — the product owner's dictated description is a simplification, not an intentional design choice.**

A Workout already models multi-set exercise entries (e.g. "4 sets × 8 reps," `domain-model.md` → Workout, and `CLAUDE.md`'s own worked example). The dictated description ("finish an exercise, rest, move to the next exercise") only mentions transitions *between* exercises, with no explicit handling of a multi-set entry. Read literally, it would mean one continuous active block per exercise regardless of how many sets it has, with a single rest before the next exercise — which cannot be reconciled with ADR-003's "set-by-set data (reps, weight, duration actually done)" being plural: there'd be no point at which four separate sets' worth of actual reps/weight for one entry get individually logged. Resolved: the live flow steps through each entry's sets one at a time — active period, then rest, then the next set of the same entry, or the next entry if that was the last set. Rest reuses the single per-exercise-entry planned value already established in `domain-model.md` → Workout (also resolving that document's long-open "is rest time per-exercise or per-set?" question). The very last set of the very last entry needs no trailing rest. See `domain-model.md` → Workout Session, "Live execution flow."

**2. Progress is measured in sets; a session can end before covering its whole plan (ended early), and this is a legitimate, persisted outcome — resolving `domain-model.md`'s open "can a session be partially completed?" question.**

Given rule 1 above, sets are the natural unit for an overall progress percentage (total sets logged / total sets planned in the pinned Workout Version), not exercises — this also keeps the progress bar moving continuously through a multi-set exercise rather than only at exercise boundaries. Separately: nothing about the live flow, or about training in general, guarantees a user always finishes what they started (time runs out, the gym closes, an injury). Forcing a binary of "completed the entire plan" or "no record exists at all" would mean losing exactly the kind of real, partial data `vision.md` commits to capturing ("the gap between planned and actual execution becomes visible and analyzable, not lost"). Resolved: a session may end before reaching the end of its plan; this is recorded as a distinct outcome — **ended early** — as opposed to **completed** (reached the end of the plan). Both are valid, permanently persisted records; ADR-004 applies identically to either. See `domain-model.md` → Workout Session, "Progress and completion."

**3. The live clocks are ephemeral UI state, with one exception; ADR-003's "set-by-set data" is unchanged by this feature, plus a session gets its own start/completion timestamps.**

The count-up "doing the exercise" timer and the count-down rest timer are pacing aids, not data the session needs to retain once a set is logged — except for a **duration-based** Set (e.g. a held plank), where the count-up timer's stopped value *is* the actual duration being recorded, not a separate ephemeral thing. ADR-003's synced set-by-set data stays exactly reps/weight/duration, unchanged by this feature. What this feature does newly confirm as needed: a session-level start timestamp and completion timestamp (session began / session ended), simply so a session can be placed in time at all — this was implicit before but is made concrete by having a real start-session action now. A per-set completion timestamp is **not** established as required this iteration. See `domain-model.md` → Workout Session, "What the live clocks are/aren't."

**4. Actual rest taken is persisted per set — reverses this document's original "UI-only" draft position, per explicit product-owner decision (2026-07-25).**

The draft position recommended treating an extended rest period as UI-only, since no described Progress Tracking capability needs rest-adherence data. Overridden by the product owner on a broader principle: **persist all of a session's actual data in full, distinct from the plan, wherever it can legitimately diverge** — not narrowly scoped to whichever fields Progress Tracking happens to consume today. Resolved: each set's actual rest duration taken (reflecting any "+seconds" adjustments) is recorded alongside that set's actual reps/weight/duration, distinct from the entry's planned rest value. See `domain-model.md` → Workout Session, "Actual rest taken," and Set, "Actual rest taken."

**Rejected**: implementing the product owner's dictation fully literally — one continuous active timer per exercise (regardless of set count) and a single rest before the next exercise. Rejected for the reasons in rule 1: it contradicts `CLAUDE.md`'s own multi-set worked example and is incompatible with ADR-003's set-by-set logging expectation.

**Process note (Challenger review, `../../scratchpad/challenger-review-iteration-4.md` §4)**: rule 1 directly overrides the product owner's own dictated words, more directly than rule 4's silent-gap-filling above — yet only rule 4 was flagged for explicit confirmation. The reasoning for rule 1 is sound (it isn't optional given what's already accepted elsewhere), but per the same "recommend explicit confirmation" treatment given rule 4, the product owner should also explicitly confirm rule 1's per-set stepping rather than have it stand as inferred intent alone.

**Confirmed by the product owner (2026-07-25)**, with a refinement beyond what this document originally proposed: per-set rest is correct, and additionally — a same-exercise transition may use an explicit, distinct "between-sets" rest value if the entry defines one (falling back to the entry's default otherwise); a transition into a *different* exercise uses the **upcoming** entry's own rest value, not the one just finished. See `domain-model.md` → Workout, "Rest granularity" (revised) and Workout Session, "Live execution flow" (revised) for the resolved rule.

### Open questions for the Architect

1. **Superset/Circuit live-execution semantics** — not addressed by the flow resolved above, which is confirmed only for a simple, linear sequence of exercise entries. A Superset (no rest between its exercises) and a Circuit (rounds of a sequence) both need their own live-stepping rule before the Training Execution aggregate can represent them mid-session; not blocking if the first build targets simple linear Workouts only.
2. **Whole-entry skip vs. session-level early termination** — the flow above only resolves *stopping the session* at whatever point is reached (rule 2). Whether a user can explicitly skip one entry outright and continue with a later one — a non-linear progression — is unaddressed and shapes whether session progression state needs to be a simple linear pointer or something more flexible.
3. **Multi-day pause/resume** — is a session interrupted and picked back up after a meaningful real-world gap (e.g. the next day) a resumption of the same session, or the start of a new one? Related: how long should an in-progress/"ended early" session realistically stay resumable (this also touches ADR-003's client-side cache lifetime, not a pure domain question) before it's simply treated as ended? Not resolved here — recommend a default be picked explicitly rather than left implicit in client behavior.
4. **Dangling Exercise reference within a pinned Workout Version, now made concrete** — this is not a new question; it sharpens `requirements-and-open-items.md`'s existing "Cross-context reference integrity" open question 2 (session-start behavior with a Dangling Reference). This feature adds two concrete sub-questions to it: does an unresolvable entry count toward the total-sets progress denominator, and can a session carrying one ever reach **completed** rather than being permanently capped at **ended early**? Recommend resolving together with that pre-existing item rather than independently.
~~5. Whether to persist actual rest duration~~ — **Resolved, product-owner confirmation 2026-07-25**: yes, persist actual rest taken per set, as part of a general "capture all actual session data, not just what today's known consumers need" principle. See rule 4 (revised) and `domain-model.md` → Workout Session/Set, "Actual rest taken." The Training Execution aggregate's Set shape must include it from the first build.

## Recommendation for next iteration

_Iteration 1_: every originally-flagged open item is now either resolved (ADR-001 through ADR-004, plus the domain-model additions for Exercise ownership/hierarchy, Set, Enrichment promotion, and Media Resource) or explicitly deferred as a deliberate scope choice (body metrics/goals, units/localization, notifications, monetization — see `../../scratchpad/open-questions/iteration-1.md`).

_Iteration 2_: the cross-context reference integrity question (above) is resolved at the business-rule level for its two triggering instances (Exercise↔Workout, both directions). Four items are now handed to the Architect (see "Open questions for the Architect," above) — none block further product analysis; they block the concrete implementation of Exercise deletion's safeguard and Workout creation's validation, which is Architect/service-loop scope from here.

_Iteration 3_: two independent gaps are resolved at the requirement level and handed to the Architect. Authenticated request identity enforces an already-decided concept (ADR-001 ownership) rather than introducing a new one — no domain-model change, two open questions for the Architect (token/failure-handling mechanism; whether shared-library reads stay anonymous). Read-after-write freshness is confirmed as a hard correctness requirement (a user's own writes must be visible to themselves on the next read) scoped to the acting user, not a system-wide instant-consistency requirement — a defect fix against FT-001's intended behavior, not a new open item.

_Iteration 4_: the live workout-session execution flow (real-time exercise/rest timers, progress, completion) is resolved at the business-rule level. Rest and progress granularity is set-by-set, not exercise-by-exercise, resolving `domain-model.md`'s long-open "per-exercise or per-set" rest question; a session may now legitimately end before covering its whole plan ("ended early"), resolving the "can a session be partially completed?" open item. The live count-up/count-down clocks are confirmed ephemeral UI state (except a duration-based Set's own duration value), and a session gains its own start/completion timestamps. Extending an in-progress rest period is treated as a UI-only convenience, not new persisted data — flagged as this iteration's closest-to-arbitrary call, worth explicit product-owner confirmation. Five items are handed to the Architect (see "Open questions for the Architect," above) — none block further product analysis; they shape how the Training Execution aggregate models in-flight progression state (Superset/Circuit stepping, non-linear skip, multi-day resume, Dangling Reference interaction, and whether to reserve room for actual-rest-taken).
