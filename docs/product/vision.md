# Forma — Product Vision

_Analyst output — iteration 1. Source: `CLAUDE.md` product description._

## What Forma is

Forma is a platform for managing a person's **training lifecycle** — from defining what a movement *is*, through planning what to do and when, to recording what actually happened and understanding progress over time.

## Why it exists

Training data today tends to live in three disconnected places: a mental/paper plan, a workout tracked loosely in a generic notes or spreadsheet app, and vague memory of "how it's going." Forma's value proposition is connecting these into one model, so that:

- plans (Workouts, Routines) and reality (Workout Sessions) are both captured, and kept **distinct** rather than conflated;
- the gap between planned and actual execution becomes visible and analyzable, not lost;
- progress tracking is a natural consequence of consistent recording, not a bolted-on feature.

## The core lifecycle

```
Exercise → Workout → Routine → Workout Session → Progress Tracking
```

- **Exercise** — a reusable definition of a movement (the "vocabulary").
- **Workout** — a reusable plan composed of exercises (the "sentence").
- **Routine** — how workouts are organized over time (the "schedule").
- **Workout Session** — what actually happened when a workout was performed (the "record").
- **Progress Tracking** — what can be learned from an accumulation of sessions (the "insight").

See [domain-model.md](domain-model.md) for the detailed shape of each concept, and [glossary.md](glossary.md) for term definitions.

## Design values carried into every iteration

- **Plan vs. reality is a first-class distinction**, not an implementation detail — this shows up throughout the domain model (see Workout vs. Workout Session in `domain-model.md`).
- **AI enrichment is an enhancement layer, not the foundation.** The core domain (exercises, workouts, routines, sessions) must be coherent and usable without any AI involvement; enrichment (muscle groups, progressions, safety notes, etc.) augments it but lives at arm's length (see `domain-model.md` → Enrichment).
- **Progress tracking is a design constraint from day one**, even though it may not ship in an MVP — every upstream concept (Exercise, Workout, Workout Session) should be shaped so that later analytics are possible without redesigning history.

## Explicitly not yet defined

`CLAUDE.md` describes the domain but does not yet define **who the user is** (a solo athlete tracking themselves? a coach managing athletes? both?), nor the platform's business model. These are treated as open questions — see [requirements-and-open-items.md](requirements-and-open-items.md) — rather than assumed.
