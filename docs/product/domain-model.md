# Forma — Domain Model (Iteration 1)

_Analyst output. Business language only — no persistence, API, or service-boundary implications here; those are the Architect's job (see `../architecture/bounded-contexts.md`)._

## Confirmed concepts (explicitly described in `CLAUDE.md`)

### Exercise

A reusable definition of a training movement. Not tied to any one workout.

- Attributes: name, description, execution instructions, required equipment, media resources, tags, difficulty.
- Enrichment (see below) may add: muscle groups, movement pattern, difficulty classification, progressions, regressions, alternative exercises, common mistakes, safety recommendations.
- **Open**: is an Exercise global/shared (a library) or can users define their own private exercises? Not stated — see open items.

### Workout

A reusable training plan composed of exercises, describing *intended* structure: sequence, reps, duration, sets, weight, rest time, and grouping constructs (supersets, circuits).

- A Workout is a template — it does not itself represent a specific occurrence in time.
- **Open**: can the same Exercise appear more than once in a Workout (e.g. warm-up set at lower weight, then working sets)? Is rest time per-exercise or per-set?

### Routine

Organizes Workouts over time — which workouts happen, when, and how often (e.g. a weekly pattern: Monday → Upper Body, Wednesday → Lower Body).

- Explicitly must **reference** Workouts, not duplicate their detail — i.e. editing a Workout should be reflected wherever it's referenced by a Routine, not require the Routine to be updated separately.
- **Open**: is a Routine's schedule a repeating pattern (e.g. "every Monday") or bound to actual calendar dates? Can rest days be modeled explicitly? What happens when a Routine changes after sessions have already been logged against it?

### Workout Session

The record of an actual performance of a Workout by a user. `CLAUDE.md` is explicit that **planned execution and actual execution must be structurally distinct** — a session doesn't just "check off" a workout, it records its own independent set-by-set data (reps, weight actually done), which may differ from the plan.

- **Open**: can a session happen without being tied to a Workout/Routine at all (an ad hoc/freestyle session)? Can a session be partially completed? Can it be edited after the fact, and if so, does that affect Progress Tracking that already consumed it?

### Progress Tracking

Derived from the accumulation of Workout Sessions over time. Explicitly a future-facing concern: progression history, personal records, training volume, consistency, performance trends, recommendations.

- Not itself a data-entry concept — it's a *read model* / analysis over Workout Sessions (and possibly body metrics — see open items).
- `CLAUDE.md` states this must influence current design even pre-MVP: whatever shape Workout Session data takes needs to support these future computations without redesign.

## Concepts implied but not yet defined by `CLAUDE.md`

These are surfaced by the Analyst as likely necessary, but are **not decided** — see `requirements-and-open-items.md`.

- **User / Athlete** — every workflow described ("performed by a user") presupposes an actor, but no identity, account, or profile concept is described at all (auth, personal data, body metrics, goals).
- **Set** — both Workout and Workout Session talk about "sets" with reps/weight; this may deserve to be a first-class concept rather than an inline attribute, especially since planned vs. actual sets must be compared.
- **Equipment** — currently just a free attribute of Exercise ("required equipment"); unclear if it needs to be its own referenceable concept (e.g. for filtering "what can I do with what I have").
- **Muscle Group / Movement Pattern** — named as possible *enrichment* outputs, but not defined as domain concepts with their own identity (would enrichment attach free text, or reference a controlled vocabulary?).
- **Enrichment (AI)** — `CLAUDE.md` mandates it stay separated from the core domain, but doesn't yet describe *how* (a separate record attached to an Exercise? a versioned suggestion pending human acceptance? fully automatic?).
- **Coach / Trainer** — not mentioned at all. Whether Forma supports anyone directing another person's training (assigning Routines, reviewing Sessions) is entirely open and materially affects the domain model if "yes."

## Relationships (as currently understood)

```
Exercise  ──(composed into, N:M, with Workout-specific parameters)──▶  Workout
Workout   ──(referenced by, N:M)───────────────────────────────────▶  Routine
Workout   ──(performed as)──────────────────────────────────────────▶  Workout Session
Routine   ──(schedules occurrences of)───────────────────────────────▶ Workout Session  (open: direct, or always via Workout?)
Workout Session ──(aggregated into)─────────────────────────────────▶ Progress Tracking
```

Note the Workout↔Workout-Session relationship needs clarification: does a session always trace back to a specific Workout (even if performed "off script"), or can it exist independently? See open items.
