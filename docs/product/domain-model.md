# Forma — Domain Model (Iteration 1)

_Analyst output. Business language only — no persistence, API, or service-boundary implications here; those are the Architect's job (see `../architecture/bounded-contexts.md`)._

## Confirmed concepts (explicitly described in `CLAUDE.md`)

### Exercise

A reusable definition of a training movement. Not tied to any one workout.

- Attributes: name, description, execution instructions, required equipment, media resources (see **Media Resource**, below), tags, difficulty.
- Enrichment (see below) may add: muscle groups, movement pattern, difficulty classification, progressions, regressions, alternative exercises, common mistakes, safety recommendations.
- **Ownership (decided)**: both a centralized/shared Exercise library and individually user-defined private Exercises exist. A private Exercise is visible only to the user who defined it. A mechanism for a user to promote a private Exercise into the shared library is deliberately deferred to a future iteration — not part of this scope.
- **Hierarchy (decided)**: an Exercise may declare a **parent** Exercise, forming a generalization/specialization relationship — e.g. "Bench Press" as a general parent, with "Barbell Bench Press" and "Dumbbell Bench Press" as specializations (children). This resolves the earlier open question on exercise variation/parameterization: variants are modeled as related Exercises via this hierarchy, not as one Exercise with an equipment parameter. Cross-visibility interaction (can a private Exercise specialize a shared one, or vice versa) is not yet specified — flagged for a future iteration.
- **Deletion while referenced by a Workout (decided, iteration 2)**: an Exercise cannot be deleted while at least one Workout still references it — the same rule for both a shared and a private Exercise. See `requirements-and-open-items.md` → "Cross-context reference integrity" for the reasoning and the ownership-model nuance this surfaces (a private Exercise's referencing Workouts always belong to the deleting user; a shared Exercise's may not).

### Workout

A reusable training plan composed of exercises, describing *intended* structure: sequence, reps, duration, sets, weight, rest time, and grouping constructs (supersets, circuits).

- A Workout is a template — it does not itself represent a specific occurrence in time.
- **Versioned (decided, [ADR-002](../architecture/adr/ADR-002-workout-versioning-and-session-snapshot.md))**: editing a Workout's structure or parameters creates a new immutable version rather than mutating the existing one in place. Prior versions remain intact.
- **Exercise reference validity (decided, iteration 2)**: a Workout references an Exercise by identity only (no duplication). Creation/editing should make a best-effort check that a referenced Exercise exists, but this is a UX safeguard, not a hard precondition for saving. Wherever a Workout's Exercise reference no longer resolves (a **Dangling Reference** — see glossary), it must be presented clearly as unavailable/removed, never silently dropped or erroring. See `requirements-and-open-items.md` → "Cross-context reference integrity."
- **Rest granularity (decided, iteration 4; refined, product-owner confirmation 2026-07-25)**: each exercise entry carries its own planned rest value, used two ways — (1) between consecutive sets of that same entry (the default), and (2) as the rest taken to *enter* that entry from whatever preceded it, i.e. the transition between two different exercises uses the **upcoming** entry's own rest value, not the one just finished ("you rest according to what's coming next, not what you just did"). An entry may additionally define a distinct, explicit **between-sets** rest value that overrides its own default specifically for repeating itself — if not provided, the entry's single rest value covers both roles. This resolves the Challenger's flagged v1 limitation (`../../scratchpad/challenger-review-iteration-4.md` §5, uniform rest regardless of transition type) directly, without inventing a third concept: attribution of inter-entry rest to the *incoming* entry, plus an optional intra-entry override, covers the case the Challenger raised (moving to different equipment plausibly wanting a longer rest — modeled by that next entry's own value being set higher, or its explicit between-sets override being set differently). See Workout Session, below, for how this plays out live.
- **Open**: can the same Exercise appear more than once in a Workout (e.g. warm-up set at lower weight, then working sets)?

### Routine

Organizes Workouts over time — which workouts happen, when, and how often (e.g. a weekly pattern: Monday → Upper Body, Wednesday → Lower Body).

- Explicitly must **reference** Workouts, not duplicate their detail — i.e. editing a Workout should be reflected wherever it's referenced by a Routine, not require the Routine to be updated separately.
- **Reference semantics (decided, [ADR-002](../architecture/adr/ADR-002-workout-versioning-and-session-snapshot.md))**: a Routine references a Workout live — it always tracks the latest version, not a version pinned at the time the Routine was created. Editing a Workout therefore changes what every Routine referencing it means going forward, without requiring the Routine itself to be touched. Historical meaning for already-executed occurrences is preserved instead at the Workout Session level (see below), not by the Routine.
- **Open**: is a Routine's schedule a repeating pattern (e.g. "every Monday") or bound to actual calendar dates? Can rest days be modeled explicitly?

### Workout Session

The record of an actual performance of a Workout by a user. `CLAUDE.md` is explicit that **planned execution and actual execution must be structurally distinct** — a session doesn't just "check off" a workout, it records its own independent set-by-set data (reps, weight actually done), which may differ from the plan.

- **Snapshot semantics (decided, [ADR-002](../architecture/adr/ADR-002-workout-versioning-and-session-snapshot.md))**: when a session starts, it pins the specific Workout **version** current at that moment. That reference never changes afterward, even if the Workout is edited later — so a session's meaning is fixed the instant it begins, independent of both future Workout edits and the Routine's live reference.
- **Offline logging (decided, [ADR-003](../architecture/adr/ADR-003-offline-workout-session-logging.md))**: while training, set-by-set data is cached locally on the client and synced to the server once connectivity returns — the one area of Forma required to tolerate offline use.
- **Historical mutability (decided, [ADR-004](../architecture/adr/ADR-004-progress-tracking-not-retroactively-recomputed.md))**: a completed session can be edited or deleted after the fact. Doing so does **not** retroactively recompute Progress Tracking values already derived from it — only future computations reflect the change.
- Can carry attached **Media Resource**s (e.g. a video of the user's own performance) — see below.
- **Live execution flow (decided, iteration 4; confirmed by product owner 2026-07-25)**: a session steps through the pinned Workout Version's exercise entries in sequence, and *within* each entry, through its sets one at a time — not once per exercise. The pattern per set is: an active period (timed, counting up, while the user performs that set), followed by a rest period (timed, counting down) before the next set begins. The rest value used is: that entry's between-sets rest value (if it has an explicit one) or its default rest value, for a same-entry transition; the **next** entry's own rest value, for the transition into a new exercise (see Workout, above → "Rest granularity"). The very last set of the very last entry needs no trailing rest — the session can complete as soon as it's logged. This resolves the ambiguity in a description that only mentions rest "between exercises": rest actually happens between every set, and a multi-set exercise entry is never collapsed into one continuous active block. **Deliberately out of scope this iteration**: how a Superset or Circuit's grouped exercises would step through this same active/rest pattern is not addressed — the flow above is confirmed only for a simple, linear sequence of exercise entries (see "Open," below).
- **Progress and completion (decided, iteration 4 — resolves "can a session be partially completed?", below)**: overall session progress is the count of sets actually logged against the total planned sets in the pinned Workout Version — sets, not exercises, are the unit progress is measured in, matching the unit the live flow above steps through. A session is **not** required to reach the end of its plan to be a valid, persisted record: it may end before covering every entry, and this is recorded as a distinct, legitimate outcome — **ended early** — rather than being blocked or silently discarded, consistent with the vision's commitment that actual execution, even incomplete, is more valuable captured than lost (see `vision.md`). A session that does reach the end of its plan is **completed**. Both states persist permanently, and ADR-004's rule that derived Progress Tracking values are not retroactively recomputed applies the same way to either. Whether a user can explicitly skip a whole entry outright (not just stop the session at it) and continue with a later one — as opposed to only ever stopping the whole session at the point reached — is **not** addressed by this decision; see "Open," below.
- **What the live clocks are/aren't (revised, product-owner decision 2026-07-25 — reverses the iteration-4 draft default)**: the count-up "doing the exercise" timer is ephemeral pacing UI, **except** for a Set recorded as a duration rather than a rep count (see Set, below — e.g. a held plank), where its stopped value *is* the actual data being captured. The count-down rest timer, however, **is** retained: the product owner's explicit direction is to persist all of a session's actual data, which may legitimately differ from the plan — not just reps/weight/duration, but the actual rest duration taken per set as well (see "Actual rest taken," next, which supersedes the draft "Extending rest — UI convenience" position). Beyond per-set actuals, a session records its own start and completion timestamps (when it began, and when it ended — whether completed or ended early), needed to place a session in time (e.g. for future Progress Tracking's consistency/trend views). A per-set completion timestamp remains **not** established as required this iteration.
- **Actual rest taken — persisted, per set (decided, product-owner confirmation 2026-07-25, supersedes the draft "UI-only" position)**: the general principle the product owner set: capture the session's actual data in full, distinct from the plan it may diverge from — the same plan-vs-actual philosophy already first-class for reps/weight (`vision.md`) now extends explicitly to rest. Concretely: each set's actual rest duration taken (including any time added via the "+seconds" control) is recorded alongside that set's actual reps/weight/duration, distinct from the entry's *planned* rest value (Workout, above → "Rest granularity"). The "+seconds" control itself remains a live adjustment to the ephemeral countdown in the moment — what changes is that the countdown's *final elapsed value*, once rest ends, is no longer discarded.
- **Open**: can a session happen without being tied to a Workout/Routine at all (an ad hoc/freestyle session)? ~~Can a session be partially completed?~~ **Resolved, iteration 4** — see "Progress and completion," above.
- **New open items, iteration 4**: how a Superset/Circuit's grouped exercises execute live (flagged above). Whether a whole exercise entry can be explicitly skipped mid-session, continuing with the next entry, rather than only ever stopping the session outright at the point reached. Whether a session interrupted and resumed after a meaningful real-world gap (e.g. the next day — not just briefly backgrounding the app mid-set) is a resumption of the same session or the start of a new one, and whether there's a limit on how long an "ended early"/in-progress session stays resumable before it's simply treated as ended. How a **Dangling Reference** (an Exercise the pinned Workout Version can no longer resolve) interacts with the mechanics above — does it count toward the total-sets denominator, can the session ever reach "completed" if one exists — sharpens, rather than newly raises, `requirements-and-open-items.md`'s existing open question 2 under "Cross-context reference integrity."

### Progress Tracking

Derived from the accumulation of Workout Sessions over time. Explicitly a future-facing concern: progression history, personal records, training volume, consistency, performance trends, recommendations.

- Not itself a data-entry concept — it's a *read model* / analysis over Workout Sessions (and possibly body metrics — see open items).
- `CLAUDE.md` states this must influence current design even pre-MVP: whatever shape Workout Session data takes needs to support these future computations without redesign.
- **Consistency model (decided, [ADR-004](../architecture/adr/ADR-004-progress-tracking-not-retroactively-recomputed.md))**: a computed value (PR, volume, trend) is a durable fact once produced, not a live-recalculated view — editing/deleting the session(s) it came from does not change it retroactively.

### Set

One discrete unit of work within an exercise's performance — a given number of **reps** at a given weight/duration. **Sets** count how many times the exercise block should be repeated; **reps** count how many repetitions make up one set.

- **Decided**: Set is an **inline ordered entry**, not an independently addressable/identified concept. It exists as an ordered list of `{reps, weight/duration}` embedded within its parent — the Workout version (planned) or the Workout Session (actual) — and has no identity or reference usable outside that parent.
- **Live-execution clarification (iteration 4)**: for a Set recorded as a duration (rather than reps), the count-up timer used during live session execution (see Workout Session, above) is the mechanism that produces that value — not a separate, ephemeral measurement. This doesn't change Set's decided shape, it only clarifies how the duration component gets captured during a live session.
- **Actual rest taken (product-owner confirmation 2026-07-25)**: a Set's actual (session) record additionally carries the actual rest duration taken after it, distinct from the entry's planned rest value — see Workout Session, above → "Actual rest taken."

### Enrichment (AI)

`CLAUDE.md` mandates enrichment stay separated from the core domain — confirmed: enrichment content is held as a distinct, clearly-labeled layer, never auto-merged into what defines an Exercise.

- **New requirement (decided)**: the application needs a **promotion mechanism** — a way for a user to review a specific AI-sourced suggestion and explicitly accept it, at which point it becomes part of the Exercise's standard/definitive data rather than a labeled suggestion. Until promoted, enrichment content stays visibly separate. This is a product/UX requirement for a future iteration, not a change to the core Exercise shape.

### Media Resource

A photo or video, either uploaded by the user or an external link (e.g. to a third-party instructional video). Resolves the earlier open item on Exercise media handling.

- **Decided**: attachable to two things — an **Exercise** (instructional/learning material) and a **Workout Session** (e.g. a video the user recorded of themselves performing that session). Not attachable to Workout or Routine.
- Capture/storage mechanics (upload limits, hosting, thumbnailing) are implementation detail, not addressed here.

## Concepts implied but not yet defined by `CLAUDE.md`

These are surfaced by the Analyst as likely necessary, but are **not decided** — see `requirements-and-open-items.md`.

- **User / Athlete** — every workflow described ("performed by a user") presupposes an actor. [ADR-001](../architecture/adr/ADR-001-user-model-iteration-1.md) confirms a single normal-user persona for this iteration (no coach/delegation), but account/profile details (auth, body metrics, goals) remain undefined.
- **Equipment** — currently just a free attribute of Exercise ("required equipment"); unclear if it needs to be its own referenceable concept (e.g. for filtering "what can I do with what I have").
- **Muscle Group / Movement Pattern** — named as possible *enrichment* outputs, but not defined as domain concepts with their own identity (would enrichment attach free text, or reference a controlled vocabulary?).

## Relationships (as currently understood)

```
Exercise  ──(composed into, N:M, with Workout-specific parameters)──▶  Workout (versioned)
Exercise  ──(parent of, generalization/specialization)──────────────▶  Exercise
Workout   ──(referenced by, N:M, always latest version — ADR-002)───▶  Routine
Workout   ──(pins the version current at start — ADR-002)───────────▶  Workout Session
Routine   ──(schedules occurrences of)───────────────────────────────▶ Workout Session  (open: direct, or always via Workout?)
Workout Session ──(aggregated into, computed once — ADR-004)────────▶ Progress Tracking
Exercise        ──(illustrated by)───────────────────────────────────▶ Media Resource
Workout Session ──(documented by)────────────────────────────────────▶ Media Resource
```

Note the Workout↔Workout-Session relationship still needs clarification on one point: does a session always trace back to a specific Workout (even if performed "off script"), or can it exist independently? The *version-pinning* mechanics of that reference, however, are settled — see [ADR-002](../architecture/adr/ADR-002-workout-versioning-and-session-snapshot.md).
