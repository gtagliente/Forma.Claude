# ADR-008: `training-execution-service` Datastore — RavenDB

## Status

**Accepted** (2026-07-25). The datastore choice itself was made directly by the product owner, after being shown the comparison in Context, below, following the same "product owner decides ahead of the normal cycle" pattern already used for [ADR-005](ADR-005-microservices-architecture.md)'s microservices call. This ADR formalizes that decision rather than re-deriving it.

**Challenger-reviewed** (2026-07-25, `../../../scratchpad/challenger-review-iteration-4.md`) — three findings accepted and incorporated directly into this document: (1) the aggregate sketch's `OwnerId` enforcement now requires ADR-007's pattern from this service's first build, not a deferred "once this service has its own equivalent" (§1 of the review — there is no remaining excuse for the deferral, unlike the original two services); (2) the repo-name recommendation is changed from `Forma.Executor` to `Forma.Session` (§3 — naming-collision risk with this ecosystem's own generic task/job-executor vocabulary); (3) a non-blocking, log-only reconciliation check for the client-submitted snapshot is now recommended rather than deferred with no observability commitment (§2 — see `../integration-patterns.md`). The additional sections below (aggregate sketch, repo-location recommendation) remain this Architect's own proposals from the same pass, not independently "Accepted" the way the datastore choice itself is — see their own framing.

## Context

[ADR-005](ADR-005-microservices-architecture.md) fixed the *rule* — every service, including `training-execution-service`, owns an independent datastore, with no shared database and no cross-service joins/foreign keys — but explicitly deferred every service's *concrete* technology choice as implementation detail ("Concrete technology stack and per-service API contracts — same deferral already stated in `architecture-approach.md`"). `exercise-service` and `training-planning-service` each already resolved that deferral locally, within their own service pipelines, without needing a central decision. `training-execution-service` never got that far: per `integration-patterns.md`, it has remained "still a placeholder (no `domain.md`/`architecture.md` populated yet)" — its datastore was simply never chosen.

The product owner has now requested the first real build-out of this service (live workout-session execution: active/rest timers, progress bar — `../../product/requirements-and-open-items.md` → "Live Workout Session Execution (Iteration 4)"), and made an explicit sequencing call for how to get there: **architecture-first** — this ADR, plus a first architecture pass, exists before any implementation or scaffolding begins. This document is that gate.

### Why this, unlike the other two services' datastore picks, is a central decision

A single service's datastore choice, made in isolation, is ordinarily a local, reversible decision scoped to one service — exactly the kind of thing `adr/README.md`'s bar says does *not* belong here. `training-execution-service`'s case is different in kind, not just because a human happened to get involved: the store has to correctly hold data whose *shape* is fixed by three already-accepted central ADRs, all of which cross this exact service boundary — [ADR-002](ADR-002-workout-versioning-and-session-snapshot.md)/[ADR-005](ADR-005-microservices-architecture.md)'s denormalized Workout Version snapshot, [ADR-003](ADR-003-offline-workout-session-logging.md)'s offline-sync target, and [ADR-004](ADR-004-progress-tracking-not-retroactively-recomputed.md)'s durable, non-retroactively-recomputed Progress Analytics facts. That's what earns this a place here ("affects more than one service/feature... system-wide architecture" per `adr/README.md`), not an arbitrary central intervention into what would otherwise be a purely local call.

### The comparison the product owner was shown

Three options were evaluated:

- **MongoDB** — a mature, widely-adopted document store. A schemaless JSON document is a natural fit for a `WorkoutSession` that needs to embed a denormalized copy of a Workout Version (ADR-002/ADR-005) alongside its own per-set actual-result data, without forcing either through a fixed relational schema. Functionally sound; not rejected on any technical gap.
- **Elasticsearch** — ruled out, not merely deprioritized. Elasticsearch is fundamentally a search/analytics index, not a transactional system-of-record store: it trades strong per-document consistency and durability guarantees for search/aggregation performance, and its write/versioning model isn't built around being the authoritative source of truth for a single owning entity. That's precisely the wrong profile for `WorkoutSession` data, which ADR-004 requires to be a durable, non-retroactively-recomputed fact once derived, and which ADR-003 requires to reliably accept out-of-order offline-synced writes as the record of what actually happened. Elasticsearch's aggregation strengths look superficially attractive for a future Progress Analytics capability, but optimizing the *write-side system of record* for that is solving the wrong layer's problem — if a dedicated search/analytics layer is ever justified, it would be evaluated later, as a read-side addition over the real system of record, not by making the system of record itself a search engine.
- **RavenDB** — also document-oriented, so it shares Mongo's natural fit for the denormalized-snapshot shape above, but it tipped the decision on two further grounds: it is built .NET-native (first-class C# client, LINQ-native querying — a stronger ecosystem match than Mongo's for a project where the other two Training/Exercise services are already .NET), and it ships with built-in Map/Reduce indexes computed and maintained by the database engine itself, which is a plausible foundation for a future Progress Analytics capability without needing to bolt on a separate analytics engine later.

The product owner chose **RavenDB**, weighing the .NET-native fit and the built-in Map/Reduce angle heavier than Mongo's marginally larger ecosystem/community size.

## Decision

**`training-execution-service` gets its own independent RavenDB store**, per ADR-005's one-datastore-per-service rule — no shared database, no cross-service joins or foreign keys with `training-planning-service` or any other service. This is the system-of-record datastore for `WorkoutSession` data (and, if/when Progress Analytics is ever built out as a real capability, its most likely home too — see Consequences, below).

## Alternatives considered

- **MongoDB** — a legitimate, technically sound alternative; not rejected for any functional gap. Passed over for RavenDB on ecosystem/tooling fit (native .NET/LINQ story matching the rest of this .NET-based project) and the built-in Map/Reduce indexing the product owner weighed as valuable for a future capability, not because it couldn't do the job.
- **Elasticsearch** — ruled out as unsuitable for a system-of-record store (see Context). Its analytics strengths don't offset being the wrong consistency/durability model for the authoritative record of a user's actual training data.
- **Reusing `training-planning-service`'s datastore** (shared database) — not seriously considered; would directly violate ADR-005's already-accepted independent-datastore rule for no offsetting benefit.

## Consequences

### For ADR-002 / ADR-005 (denormalized Workout Version snapshot)

RavenDB's schemaless JSON document model directly fits what those two ADRs already require: a `WorkoutSession` document embeds its own copy of the pinned Workout Version's exercise entries (sets, reps, weight, rest, grouping) in whatever shape that version had at the moment the session started, with no shared relational schema to reconcile against `training-planning-service`. Because the snapshot is captured once and never mutated afterward (ADR-002), RavenDB's per-document optimistic concurrency is sufficient here — no cross-document transaction is needed for this piece.

### For ADR-003 (offline sync target)

The server-side target for cached, client-synced session data (ADR-003) is naturally one `WorkoutSession` document per session, identified once at session-start and upserted from whatever the client eventually syncs, however late or out of order. RavenDB's document-by-id model fits this cleanly. This ADR does not resolve the sync protocol itself (batching, handling a session partially synced twice) — that stays implementation detail, the same deferral ADR-003 already made.

### For ADR-004 (durable, non-recomputed Progress Analytics facts) — a plausible future fit, not a commitment

RavenDB's built-in Map/Reduce indexes are a plausible mechanism for a future Progress Analytics capability that needs to read across many `WorkoutSession` documents (training volume, PRs, trends) while respecting ADR-004's "computed once, not retroactively recomputed" rule. This is named here only because it materially informed the product owner's choice between the three options — it is **not** a commitment to Map/Reduce indexes being the actual mechanism once that capability is designed for real. Progress Analytics remains an internal capability of `training-execution-service` with no independent structure (`bounded-contexts.md`, Context 4; `context-map.md` → "Deferred domain areas") and is not being built now — over-committing its design here, ahead of any real requirement to build it, would be exactly the premature complexity `CLAUDE.md` warns against.

### General

- Confirms `training-execution-service`'s first concrete technology decision, ahead of any implementation — the product owner's requested architecture-first sequencing for this service, satisfied by this ADR plus the sketch below.
- Concrete document/collection layout, indexing strategy, and migration approach are left to `training-execution-service`'s own future Service Architect — this ADR fixes the technology, not the schema.

## First architecture sketch: the `WorkoutSession` aggregate's shape

_Conceptual/aggregate-level only, not a field-by-field data model — that belongs to `training-execution-service`'s own future Service Architect, once the service repo exists, consistent with this repo's lean-docs convention of not mirroring per-service detail centrally. Recorded here only to the depth needed to unblock initial scaffolding. This is this Architect's own proposal from the current pass, not itself independently "Accepted" the way the datastore choice above is — it's exactly the kind of first-pass output the Challenger reviews next._

`WorkoutSession` is the aggregate root for this context (`bounded-contexts.md`, Context 3). A session document needs to hold, conceptually:

- **A denormalized plan snapshot** — the copy of the pinned Workout Version's data ADR-002/ADR-005 already require: ordered exercise entries, each with its planned sets/reps/weight/duration and single per-entry rest value (`domain-model.md` → Workout). Captured once, at session start (see "Session-start Workout Version fetch mechanism" in `../integration-patterns.md`), and never changed afterward regardless of later edits to the live Workout.
- **Per-set actual results** — a structure mirroring the plan snapshot's sets, each holding what was actually logged: reps/weight, or duration for a duration-based Set (`domain-model.md` → Set), **and the actual rest duration taken after that set** (product-owner confirmation 2026-07-25, `domain-model.md` → Workout Session/Set, "Actual rest taken" — distinct from the plan snapshot's planned rest value). Populated incrementally as the live flow steps through it (`domain-model.md` → Workout Session, "Live execution flow"), arriving via ADR-003's offline-tolerant sync.
- **Status/outcome** — in-progress while being performed; on end, either **Completed** or **Ended Early** (`domain-model.md` → "Progress and completion") — both permanent once set, per ADR-004.
- **Session-level timestamps** — started-at and ended-at, the only session-level timing data confirmed as required this iteration (no per-set completion timestamp — `domain-model.md` → "What the live clocks are/aren't").
- **Owning user** — an `OwnerId`, the same ownership shape as the other three services (ADR-001), enforced via [ADR-007](ADR-007-jwt-bearer-authentication.md)'s existing JWT bearer pattern from this service's first real build — not deferred. Unlike `exercise-service`/`training-planning-service`, which were first built before `identity-service` issued real tokens (a genuine sequencing constraint at the time), `training-execution-service` is being built after ADR-007 already exists, is Accepted, and is proven in production against the same `identity-service`. `WorkoutSession` data is at least as sensitive as the Exercise/Workout data ADR-007 already protects, and can carry attached Media Resources considerably more sensitive than shared-library content — there is no equivalent excuse to ship this service's initial build without it.

A progress percentage (sets logged / total planned sets) is a value derivable from the two structures above at read time; nothing here requires it to be separately stored — but that is exactly the kind of call left to the Service Architect, not fixed by this sketch.

**Deliberately left open, not sketched here** — each is an unresolved question from Iteration 4 (`requirements-and-open-items.md` → "Live Workout Session Execution," open questions 1-4) that would reshape this aggregate, and locking a shape around an assumed answer now would be premature: how a Superset/Circuit's grouped entries are represented mid-session; whether a whole entry can be explicitly skipped; multi-day pause/resume semantics; how a Dangling Exercise reference inside the snapshot is represented, and whether it blocks reaching "Completed"; and whether room should be reserved for an actual-rest-taken value. None of these block scaffolding a first version of the aggregate targeting simple, linear Workouts — they block extending it later.

## Proposed service name and repo location (pending product-owner confirmation — not decided by this ADR)

Not a decision this ADR makes — recorded here only so the recommendation is easy to act on once the product owner confirms it, per that same architecture-first sequencing request.

Two placement precedents already exist:

- **Standalone top-level repo** — `exercise-service` → `Forma.Exercise`, its own repo, sibling to `Forma.Claude`.
- **Subfolder inside the `Forma.Resource` monorepo, scaffolded from a reusable template** — `identity-service` → `Forma.Resource/Forma.Auth`; `training-planning-service` → `Forma.Resource/Forma.Planner`, itself produced by renaming the `Forma.Resource/Template_DDD` scaffold via `Forma.Resource/Template_DDD/Rename_2.ps1`.

The second pattern is also the one still actively provisioned for reuse: `Forma.Resource/Template_DDD` remains present and un-renamed today, and its rename script's own name (`Rename_2.ps1` — already the *second* invocation of that mechanism, after whatever produced `Forma.Planner`) reads as a scaffold deliberately kept ready for exactly this: the next new .NET service.

**Recommendation: place `training-execution-service` at `Forma.Resource/Forma.Session`, produced by renaming `Forma.Resource/Template_DDD` via its existing rename script.** Following the monorepo-subfolder pattern, not a new standalone top-level repo, because that is the pattern this project has actually repeated (twice, for `identity-service` and `training-planning-service`) since `Forma.Exercise` was first built; `Forma.Exercise`'s standalone placement reads as this project's original, one-off choice, not an established rule to keep matching going forward.

Naming: **revised from this Architect's original pick, `Forma.Executor`, to `Forma.Session`**, following Challenger review (`../../../scratchpad/challenger-review-iteration-4.md` §3). The original reasoning was word-formation consistency with `Forma.Auth`/`Forma.Planner` (role-nouns for each context's core activity — Identity/"Auth", Training Planning/"Planner", so Training Execution/"Executor"). The Challenger's objection: "Executor"/"ExecutorService" is a heavily overloaded term of art for generic task/thread/job-execution infrastructure in exactly this project's own .NET ecosystem — a developer scanning `Forma.Resource/` would have every reason to assume `Forma.Executor` is shared cross-cutting infrastructure, not the service owning live workout-session execution, unlike `Forma.Auth`/`Forma.Planner` which read correctly in context. `Forma.Session`, named directly after the aggregate root (`WorkoutSession`) the same way `Forma.Exercise` is named after its own root, avoids that collision.

**Confirmed by the product owner (2026-07-25): `Forma.Resource/Forma.Session`.** No longer pending — this is the name and location to scaffold from `Template_DDD` via `Rename_2.ps1`.

This section does not touch `Forma.Resource/Template_DDD` or create any new folder; it is a recommendation only, pending explicit product-owner confirmation before any scaffolding happens.

## Explicitly deferred (not addressed by this ADR)

- Concrete RavenDB document/collection design, index definitions, and any indexing strategy for a future Progress Analytics capability — service-loop detail, once the service repo exists.
- *How* the denormalized Workout Version copy is obtained at session start (as opposed to *that* RavenDB is what holds it) — resolved in this same central-loop pass, but recorded separately in `../integration-patterns.md` → "Session-start Workout Version fetch mechanism," not here.
- Final confirmation of the new service's name and repo location — see the section above; owned by the product owner, not this ADR.

## References

- `../bounded-contexts.md` (Context 3 — Training Execution)
- `../context-map.md`
- `ADR-001-user-model-iteration-1.md`, `ADR-002-workout-versioning-and-session-snapshot.md`, `ADR-003-offline-workout-session-logging.md`, `ADR-004-progress-tracking-not-retroactively-recomputed.md`, `ADR-005-microservices-architecture.md`, `ADR-007-jwt-bearer-authentication.md`
- `../integration-patterns.md` → "Session-start Workout Version fetch mechanism" (the companion resolution from this same central-loop pass)
- `../../product/requirements-and-open-items.md` → "Live Workout Session Execution (Iteration 4)"
- `../../product/domain-model.md` → Workout Session, Set
