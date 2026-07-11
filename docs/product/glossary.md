# Forma — Glossary (Iteration 1)

Canonical definitions, so every agent and document uses these words consistently. Terms marked **(proposed)** are Analyst suggestions, not yet confirmed.

| Term | Definition |
|---|---|
| **Exercise** | A reusable definition of a single training movement (e.g. Push Up, Deadlift), independent of any specific plan. |
| **Workout** | A reusable, named plan composed of exercises with intended parameters (sets, reps, weight, rest, sequencing). Represents *intent*, not an event. |
| **Routine** | An arrangement of Workouts over time (e.g. a weekly schedule). References Workouts rather than duplicating them. |
| **Workout Session** | The record of one actual, real-world performance of a Workout by a user — what really happened, which may differ from the plan. |
| **Planned execution** | The intended parameters of a Workout (e.g. "4 sets × 8 reps × 80kg"), as opposed to what actually happened. |
| **Actual execution** | The as-performed data captured in a Workout Session (e.g. "Set 3: 7 reps × 80kg") — always tied to a specific session, never edited retroactively into the Workout plan. |
| **Set (proposed)** | One discrete unit of work within an exercise's performance (a given number of reps at a given weight/duration). Appears in both planned (Workout) and actual (Workout Session) contexts. |
| **Superset** | Two or more exercises performed back-to-back with no rest between them, as a single grouped unit within a Workout. |
| **Circuit** | A sequence of exercises performed in rotation, typically repeated for multiple rounds, as a single grouped unit within a Workout. |
| **Progress Tracking** | Analysis derived from accumulated Workout Sessions over time (trends, records, volume, consistency) — not itself directly entered data. |
| **Personal Record (PR)** | A best-ever result for a given exercise/metric (e.g. heaviest weight, most reps), computed from Workout Session history. |
| **Training Volume** | A quantitative measure of total work done (commonly sets × reps × weight), aggregated over a period. |
| **Enrichment** | Additional Exercise information (muscle groups, movement pattern, progressions/regressions, safety notes, etc.) that may be produced by an external/AI intelligence service. Explicitly kept separate from the core domain per `CLAUDE.md`. |
| **User / Athlete (proposed)** | The person performing training and whose data (exercises used, sessions, progress) is being managed. Not yet formally defined — see open questions. |
| **Coach (proposed, unconfirmed)** | A hypothetical actor who might direct another user's training. Not mentioned in `CLAUDE.md` — existence is an open question, not an assumption. |
