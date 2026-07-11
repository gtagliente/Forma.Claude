# Forma — Architecture Approach Options (Iteration 1)

_Architect output. Proposal only — **no decision is made here**; this becomes an ADR only once explicitly accepted. Evaluated against the bounded contexts in `bounded-contexts.md` and the "avoid unnecessary complexity" principle in `CLAUDE.md`._

## Options considered

### A. Modular monolith (single deployable, module boundaries mirror bounded contexts)

- **Fit**: The bounded contexts in `bounded-contexts.md` currently show only one-directional dependencies and no evidence yet of needing independent scaling, independent deployment cadence, or independent team ownership — none of these have been established as real requirements. A modular monolith lets those module boundaries exist in code (enforcing the same one-directional dependencies) without paying for network calls, distributed transactions, or multi-service operational overhead.
- **Risk**: If module boundaries aren't actually enforced (e.g. Training Planning reaching directly into Training Execution's internals), the "module" boundary erodes over time. Requires discipline, not infrastructure.

### B. Microservices (one service per bounded context, per `CLAUDE.md`'s illustrative `docs/services/` tree)

- **Fit**: `CLAUDE.md`'s example directory tree shows `identity-service`, `exercise-service`, `workout-service`, `routine-service`, `ai-enrichment-service` — but this appears to be an *illustrative target structure*, not a stated requirement. Nothing in the Analyst's output currently justifies this: there's no described scale requirement, no described need for independent deployment, and the user/team model (solo vs. coach-athlete) that would drive real service ownership boundaries is still unresolved (see Analyst open items).
- **Risk**: Premature microservices split before contexts have stabilized means the *first* real domain refinement (e.g. discovering Set needs to be first-class, or that Routine↔Workout referencing must change) becomes a cross-service migration instead of a code change. This is exactly the kind of premature complexity `CLAUDE.md`'s "Avoid Premature Complexity" principle warns against.

### C. Single service, but no internal module boundaries at all (unstructured monolith)

- **Fit**: Fastest to start, but discards the bounded-context work already done for no benefit — the boundaries identified are cheap to express as modules and expensive to reconstruct later from tangled code.
- **Risk**: Rejected as strictly worse than Option A for the same deployment cost.

## Recommendation (proposal, not a decision)

**Option A — modular monolith**, with module boundaries matching the five contexts in `bounded-contexts.md` (Exercise Library, Training Planning, Training Execution, Progress Analytics, AI Enrichment; Identity added if/when confirmed). Revisit Option B if and when a concrete, demonstrated need emerges (e.g. independent scaling of AI Enrichment due to cost/latency isolation, or independent team ownership once the org has more than one team). This defers the microservices investment without foreclosing it — `CLAUDE.md`'s example services structure remains a valid *future* target, reachable by extracting a module once justified, rather than a starting assumption.

## Explicitly deferred (not addressed by this proposal)

- Concrete technology stack.
- Data storage choices per context.
- API contract design.
- Whether AI Enrichment's "external intelligence service" is a third-party API, a separate in-house service, or a library — all consistent with keeping it a separate module either way.
