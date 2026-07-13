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
- **Open**: can the same Exercise appear more than once in a Workout (e.g. warm-up set at lower weight, then working sets)? Is rest time per-exercise or per-set?

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
- **Open**: can a session happen without being tied to a Workout/Routine at all (an ad hoc/freestyle session)? Can a session be partially completed?

### Progress Tracking

Derived from the accumulation of Workout Sessions over time. Explicitly a future-facing concern: progression history, personal records, training volume, consistency, performance trends, recommendations.

- Not itself a data-entry concept — it's a *read model* / analysis over Workout Sessions (and possibly body metrics — see open items).
- `CLAUDE.md` states this must influence current design even pre-MVP: whatever shape Workout Session data takes needs to support these future computations without redesign.
- **Consistency model (decided, [ADR-004](../architecture/adr/ADR-004-progress-tracking-not-retroactively-recomputed.md))**: a computed value (PR, volume, trend) is a durable fact once produced, not a live-recalculated view — editing/deleting the session(s) it came from does not change it retroactively.

### Set

One discrete unit of work within an exercise's performance — a given number of **reps** at a given weight/duration. **Sets** count how many times the exercise block should be repeated; **reps** count how many repetitions make up one set.

- **Decided**: Set is an **inline ordered entry**, not an independently addressable/identified concept. It exists as an ordered list of `{reps, weight/duration}` embedded within its parent — the Workout version (planned) or the Workout Session (actual) — and has no identity or reference usable outside that parent.

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
