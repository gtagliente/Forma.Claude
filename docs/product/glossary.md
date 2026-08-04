# Forma — Glossary (Iteration 1)

Canonical definitions, so every agent and document uses these words consistently. Terms marked **(proposed)** are Analyst suggestions, not yet confirmed.

| Term | Definition |
|---|---|
| **Exercise** | A reusable definition of a single training movement (e.g. Push Up, Deadlift), independent of any specific plan. |
| **Workout** | A reusable, named plan composed of exercises with intended parameters (sets, reps, weight, rest, sequencing). Represents *intent*, not an event. |
| **Routine** | An arrangement of Workouts over time (e.g. a weekly schedule). References Workouts rather than duplicating them. |
| **Workout Session** | The record of one actual, real-world performance of a Workout by a user — what really happened, which may differ from the plan. Pins the specific Workout Version current when it started (see [ADR-002](../architecture/adr/ADR-002-workout-versioning-and-session-snapshot.md)). Ends either **Completed** (reached the end of its plan) or **Ended Early** (stopped before that) — both are valid, permanently persisted outcomes; see `domain-model.md` → Workout Session, "Progress and completion." |
| **Workout Version** | An immutable snapshot of a Workout's structure/parameters at a point in time. Editing a Workout creates a new Version rather than mutating the previous one. A Routine always tracks the latest Version (live); a Workout Session pins the Version current when it started. See [ADR-002](../architecture/adr/ADR-002-workout-versioning-and-session-snapshot.md). |
| **Planned execution** | The intended parameters of a Workout (e.g. "4 sets × 8 reps × 80kg"), as opposed to what actually happened. |
| **Actual execution** | The as-performed data captured in a Workout Session (e.g. "Set 3: 7 reps × 80kg") — always tied to a specific session, never edited retroactively into the Workout plan. |
| **Set** | One inline, ordered unit of work within an exercise's performance — a given number of Reps at a given weight/duration. Not an independently addressable concept: it exists only embedded within its parent Workout Version (planned) or Workout Session (actual). Appears in both planned and actual contexts. |
| **Reps** | The number of repetitions performed within one Set. |
| **Superset** | Two or more exercises performed back-to-back with no rest between them, as a single grouped unit within a Workout. |
| **Circuit** | A sequence of exercises performed in rotation, typically repeated for multiple rounds, as a single grouped unit within a Workout. |
| **Progress Tracking** | Analysis derived from accumulated Workout Sessions over time (trends, records, volume, consistency) — not itself directly entered data. |
| **Personal Record (PR)** | A best-ever result for a given exercise/metric (e.g. heaviest weight, most reps), computed from Workout Session history. |
| **Training Volume** | A quantitative measure of total work done (commonly sets × reps × weight), aggregated over a period. |
| **Enrichment** | Additional Exercise information (muscle groups, movement pattern, progressions/regressions, safety notes, etc.) that may be produced by an external/AI intelligence service. Explicitly kept separate from the core domain per `CLAUDE.md`, held as a labeled suggestion until promoted (see Enrichment Promotion). |
| **Enrichment Promotion** | The user-driven act of accepting a specific AI-sourced Enrichment suggestion, turning it into part of an Exercise's standard/definitive data. Until promoted, enrichment content stays visibly separate from the core Exercise. |
| **Media Resource** | A photo or video, either uploaded by a user or an external link. Attachable to an Exercise (instructional material) or a Workout Session (e.g. a video of the user's own performance). See `domain-model.md` → Media Resource. |
| **Shared Exercise** | An Exercise belonging to the centralized/curated library, visible to all users. |
| **Private Exercise** | An Exercise defined by an individual user, visible only to that user. A future iteration may add a way to promote a Private Exercise into the Shared library — not built yet. |
| **Parent Exercise / Specialization** | Exercises may relate via generalization/specialization: a more general Exercise (parent, e.g. "Bench Press") can have more specific variant Exercises (children/specializations, e.g. "Barbell Bench Press," "Dumbbell Bench Press"). |
| **User / Athlete** | The person performing training and whose data (exercises used, sessions, progress) is being managed. For this iteration, confirmed as a single "normal user" persona with no roles or delegation — see [ADR-001](../architecture/adr/ADR-001-user-model-iteration-1.md). |
| **Dangling Reference** | A reference from one concept to another (e.g. a Workout's reference to an Exercise) that no longer resolves because the referenced item was deleted. The referencing concept must present it clearly as unavailable/removed wherever shown, never silently drop it or error. See `requirements-and-open-items.md` → "Cross-context reference integrity." |

## Explicitly out of scope this iteration

Terms considered but not adopted, kept here rather than in the main table so they aren't mistaken for settled vocabulary (per Challenger review — a term sitting in the main glossary risks being casually cited as real by later iterations).

| Term | Status |
|---|---|
| **Coach** | A hypothetical actor who might direct another user's training. [ADR-001](../architecture/adr/ADR-001-user-model-iteration-1.md) decided Forma has only a single normal-user persona for this iteration — Coach is not modeled. May be revisited in a future iteration if a real requirement emerges. |
