# Forma — Context Map (Iteration 1)

_Synthesizes `bounded-contexts.md` (Architect) as challenged in `../../scratchpad/challenger-review-iteration-1.md`. Reflects the narrowed, more conservative scope the Challenger proposed for the next iteration — this file is the current best shared understanding, not a final decision._

## Domain areas

Three domain areas are treated as sufficiently justified to shape structure right now; two more are recognized but deliberately **not** given independent structure yet (see "Deferred domain areas" below), per the Challenger's over-engineering findings.

```
┌─────────────────────┐        ┌───────────────────────┐        ┌───────────────────────┐
│   Exercise Library   │──ref──▶│   Training Planning    │──perf─▶│   Training Execution    │
│ (Exercise, Equipment,│        │ (Workout, Routine)      │        │  (Workout Session,      │
│  Tags; enrichment as │        │                         │        │   incl. progress        │
│  internal sub-area)  │        │                         │        │   analysis capability)  │
└─────────────────────┘        └───────────────────────┘        └───────────────────────┘
```

- **Exercise Library** — the vocabulary of movements. Includes AI enrichment as an internal concern (one-directional: enrichment reads/annotates Exercises, never the reverse), not a separate area yet.
- **Training Planning** — turns exercises into intended plans (Workout) and schedules (Routine).
- **Training Execution** — records what actually happened (Workout Session), and, for now, also owns the progress-analysis capability computed from that data (rather than a separate area), since it has no independent state of its own yet.

## Deferred domain areas (recognized, not yet structured)

- **Progress Analytics as its own area** — revisit once it has state/consumers beyond a view over Training Execution data.
- **AI Enrichment as its own area** — revisit once the "external intelligence service" has concrete integration concerns (async jobs, retries, cost isolation) that outgrow living inside Exercise Library.
- **Identity** — existence itself unconfirmed (solo-user vs. coach-athlete model unresolved). Not drawn as a box on this map at all, to avoid anchoring on an unconfirmed concept, per Challenger's note on the glossary's "Coach" term. Every area above implicitly needs "a user" to scope data to, but who/what that means is an open question, not a modeled context.

## Possible service boundaries — and why none are assumed yet

The three domain areas above are natural seams *if and when* the system is ever split into independently deployable services. Today, nothing in the product/domain understanding justifies that split (see `architecture-approach.md`):

- No described requirement for independent scaling of any one area.
- No described requirement for independent deployment cadence.
- No described multi-team ownership model.

**Alternative evaluated and currently preferred**: a **modular monolith**, where these three domain areas are enforced as internal module boundaries (one-directional dependencies: Exercise Library ← Training Planning ← Training Execution) inside a single deployable. This preserves the option to extract any module into a real service later — the boundary already exists in code — without paying for distributed-system complexity now.

Should service extraction ever be justified, `Exercise Library`, `Training Planning`, and `Training Execution` are the natural first candidates for separate services, in that order of likely independence (Exercise Library changes least often and has the fewest dependencies).

## Relationships between concepts (summary)

```
Exercise Library  ──referenced by (identity only, no duplication)──▶  Training Planning
Training Planning ──performed as (snapshot at start — proposal, unconfirmed)──▶  Training Execution
Training Execution ──aggregated internally into──▶  Progress analysis (capability, not yet its own area)
```

See `../product/domain-model.md` for the underlying business-level relationships this map is built from.

## Status

Proposal, reflecting Iteration 1 discussion including Challenger input. Not an ADR. To be revisited once the user/coach question (see `../../scratchpad/open-questions/iteration-1.md`) and the snapshot-vs-live-reference question are resolved, as both may reshape this map.
