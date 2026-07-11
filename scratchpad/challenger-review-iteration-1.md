# Challenger Review — Iteration 1

_Reviews: `../docs/product/*` (Analyst) and `../docs/architecture/bounded-contexts.md` + `architecture-approach.md` (Architect)._

## What holds up well

- The Architect's recommendation to defer microservices (Option A, modular monolith) is well-justified and consistent with `CLAUDE.md`'s "avoid premature complexity" principle — no scale, team, or deployment-cadence requirement has actually been established. Endorsed.
- The Analyst correctly refused to invent a user/coach persona rather than silently assuming one. That discipline should continue.
- Explicit snapshot-vs-live-reference framing for Workout→Workout Session is a real, well-identified fork rather than a hand-waved detail.

## Questionable assumptions

1. **"All context dependencies are one-directional" is asserted too confidently.** `bounded-contexts.md` draws a clean acyclic graph, but the Analyst's own domain model lists "recommendations" as a future Progress Tracking capability (`domain-model.md` → Progress Tracking). A recommendation ("try adding a 5th set") that feeds back into Training Planning *is* a reverse dependency. The Architect should either explicitly scope recommendations out of this graph (fine, if stated) or acknowledge the graph is provisional, not proven acyclic.
2. **The Training-Execution-snapshots-the-plan design is presented as a near-decision inside a document titled "proposal."** It has real product consequences (a user editing a Workout won't retroactively change what old, or even in-progress, sessions show) that the Analyst hasn't validated as desired behavior. This should go back to the Analyst/product owner as a question, not be adopted by default because it's architecturally convenient.
3. **Defining "Coach" in the glossary, even flagged unconfirmed, risks anchoring later iterations.** A term that exists in the glossary tends to get treated as real by agents skimming for context. Recommend either removing it until confirmed, or marking it much more aggressively (e.g. a separate "rejected/unconfirmed terms" section) so it can't be casually cited as settled vocabulary.

## Over-engineering risks

1. **Six candidate bounded contexts (five confirmed + one contingent) is a lot of structure for a product with zero confirmed users and no shipped MVP.** Two of them look premature as *separate* contexts today:
   - **Progress Analytics** — currently has no data of its own beyond what Training Execution already owns, and `CLAUDE.md` says tracking should influence design, not that it must be architecturally separated now. Consider: fold it into Training Execution as a capability for now, and split it out only once it grows real state (e.g. materialized aggregates, its own storage) or its own consumers beyond a single UI view.
   - **AI Enrichment** — `CLAUDE.md`'s requirement is that it stays *separated from the core domain*, which is satisfiable with a one-directional dependency and a clearly separated data shape inside the Exercise Library module; it doesn't necessarily require its own bounded context from day one. Promote it to a full context once the "external intelligence service" is concrete enough to have its own integration concerns (retries, async jobs, cost/latency isolation).
   - Net suggestion: start iteration 2 with **3 contexts** (Exercise Library [incl. enrichment as an internal sub-module], Training Planning, Training Execution [incl. analytics as a capability]) and let Progress Analytics / AI Enrichment earn their independence.
2. **Elevating "Set" to a first-class domain concept** (Analyst's proposal) is reasonable but not yet justified by a concrete need beyond "it's mentioned in both places." Fine to keep as a proposal; just flagging it shouldn't be treated as settled before the Architect designs storage around it.

## Missing concepts / gaps

- **Body metrics & goals** are mentioned only in passing (Analyst's missing-requirements list, Architect's Progress Analytics note) but never actually assigned an owner, even provisionally. If Progress Tracking is meant to relate performance to goals, this gap will resurface soon — worth a dedicated question rather than a footnote (added to open questions).
- **Exercise variation/parameterization** (e.g. "Barbell Bench Press" vs. "Dumbbell Bench Press" — same movement pattern, different equipment) isn't addressed: are these separate Exercises, or one Exercise parameterized by equipment? This affects both the Exercise Library model and how Workouts reference exercises.
- **Historical mutability**: what happens to a Workout Session (and anything computed from it) if it's edited or deleted after the fact — not addressed by either agent yet, only implicitly touched by the Analyst's open items.

## Simplifications proposed for next iteration

- Shrink iteration-2 architecture scope to 3 contexts (see above) instead of 5–6.
- Resolve the user/coach question *before* spending more Architect effort on Identity, since almost every other boundary decision is cheap to revisit but Identity's shape is not.
- Treat "Set as first-class concept" and "snapshot vs. live reference" as two explicit product-decision questions to route back to the Analyst/human, not architectural defaults.
