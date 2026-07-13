# Forma - Project Intelligence Context

## Project Purpose

Forma is a workout management platform.

The objective of the project is not only to build an application, but to progressively build a reliable domain understanding and an evolvable software architecture through iterative analysis.

Before implementing features, the system must develop a clear understanding of:

* user needs;
* business concepts;
* domain boundaries;
* architectural constraints;
* implementation trade-offs.

The project should evolve from domain understanding rather than from premature technical decisions.

---

# Product Vision

Forma allows users to manage their training lifecycle.

The core workflow is:

```
Exercise
    ↓
Workout
    ↓
Routine
    ↓
Workout Session
    ↓
Progress Tracking
```

The platform should allow users to:

* create and manage exercises;
* compose exercises into workouts;
* organize workouts into routines;
* execute training sessions;
* record performed exercises;
* analyze progress over time.

---

# Core Domain Concepts

## Exercise

An exercise represents a reusable training movement.

Examples:

* Push Up
* Pull Up
* Squat
* Deadlift

An exercise may contain:

* name;
* description;
* execution instructions;
* required equipment;
* media resources;
* tags;
* difficulty.

The system may later enrich exercises using external intelligence services.

Possible enrichments:

* involved muscle groups;
* movement pattern;
* difficulty classification;
* progressions;
* regressions;
* alternative exercises;
* common mistakes;
* safety recommendations.

AI enrichment must remain separated from the core domain.

---

## Workout

A workout represents a reusable training plan composed of exercises.

A workout defines how exercises should be performed.

Possible workout elements:

* exercises sequence;
* repetitions;
* duration;
* number of sets;
* weight;
* rest time;
* supersets;
* circuits.

Example:

```
Workout: Upper Body Strength

Exercise:
Pull Up

Sets:
4

Repetitions:
8

Rest:
120 seconds
```

The workout represents the intended training structure.

---

## Routine

A routine represents the organization of workouts over time.

A routine defines:

* which workouts should be performed;
* when they should happen;
* repetition frequency.

Examples:

```
Weekly Routine

Monday:
Upper Body

Wednesday:
Lower Body

Friday:
Upper Body
```

A routine should reference workouts but should not duplicate workout details.

---

## Workout Session

A workout session represents the real execution of a workout by a user.

The system must clearly separate:

Planned execution:

```
Bench Press
4 sets
8 repetitions
80kg
```

Actual execution:

```
Set 1:
8 repetitions
80kg

Set 2:
8 repetitions
80kg

Set 3:
7 repetitions
80kg

Set 4:
8 repetitions
75kg
```

This distinction is fundamental for future tracking and analytics.

---

## Progress Tracking

Progress tracking is generated from workout sessions.

Future capabilities may include:

* progression history;
* personal records;
* training volume;
* consistency;
* performance trends;
* recommendations.

Tracking should influence current design decisions even if it is not part of the first MVP.

---

# Initial Product Analysis Goal

The first objective is not implementation.

The first objective is to perform a complete domain analysis.

The analysis should identify:

* business concepts;
* user workflows;
* domain rules;
* missing requirements;
* possible bounded contexts;
* aggregate boundaries;
* future evolution points.

The result should be a stable understanding of the product before implementation begins.

---

# Multi-Agent Analysis Process

The project uses three specialized agents.

The agents should work iteratively.

The process is:

```
Analyst
    ↓
Architect
    ↓
Challenger
    ↓
Knowledge Update
    ↓
Next Iteration
```

---

# Analyst Agent

The Analyst focuses on product and domain understanding.

Responsibilities:

* analyze user needs;
* identify business concepts;
* define workflows;
* discover missing requirements;
* improve domain terminology.

The Analyst must avoid technical implementation decisions.

Output:

* domain analysis;
* requirements;
* use cases;
* business rules.

---

# Architect Agent

The Architect transforms domain understanding into technical structure.

Responsibilities:

* identify bounded contexts;
* define system boundaries;
* identify aggregates;
* evaluate architecture approaches;
* analyze dependencies;
* identify technical risks.

Possible areas of analysis:

* modular monolith;
* microservices;
* service boundaries;
* APIs;
* events;
* data ownership.

The Architect must avoid unnecessary complexity.

Every architectural decision must be justified by a real requirement.

Output:

* architecture proposals;
* bounded contexts;
* ADR candidates;
* technical constraints.

---

# Challenger Agent

The Challenger reviews previous conclusions.

Responsibilities:

* challenge assumptions;
* identify over-engineering;
* suggest simpler alternatives;
* detect missing functionality;
* identify future risks.

The Challenger should ask:

* Are we solving a real problem?
* Is this complexity necessary now?
* Can this decision be postponed?
* Are boundaries correctly defined?

Output:

* improvements;
* risks;
* rejected alternatives;
* simplifications.

---

# Decision Principles

The project follows these principles:

## Domain First

Understand the problem before choosing technology.

## Avoid Premature Complexity

Do not introduce complexity without a demonstrated need.

## Evolution Over Perfection

The architecture should support future evolution without requiring unnecessary initial investment.

## Explicit Decisions

Important decisions must be documented.

## Persistent Knowledge

Important knowledge must exist outside conversations.

---

# Knowledge Management

The repository is the long-term memory of the project.

Conversation context is temporary.

All important decisions and discoveries must be persisted in project documentation.

Future agents should use repository knowledge instead of relying only on previous conversations.

---

# Documentation Structure

Knowledge here is kept deliberately flat. Per-service, per-feature, and per-branch
implementation detail lives with the code that implements it — in that service's own
repo (e.g. `Forma.Exercise`, `Forma.Resource`) — not mirrored here. This repo only
holds knowledge that's genuinely global to the whole product/system:

```
Forma/

docs/
│
├── README.md
│
├── product/                               product vision, domain model, glossary,
│   ├── vision.md                          requirements — the shared domain
│   ├── requirements-and-open-items.md     understanding, in business language.
│   ├── domain-model.md
│   └── glossary.md
│
├── architecture/                          system-wide architecture: options
│   ├── architecture-approach.md           considered, bounded contexts, the
│   ├── bounded-contexts.md                context map, integration patterns
│   ├── context-map.md                     between services — plus the durable
│   ├── integration-patterns.md            decision log:
│   └── adr/                               (index: unbounded — one ADR per decision)
│       ├── README.md
│       └── ADR-NNN-short-name.md
│
└── engineering/
    ├── coding-standards.md
    ├── git-workflow.md
    ├── testing-strategy.md
    └── devops.md

.claude/agents/ (repo root, not under docs/) — live Claude Code subagents (analyst,
architect, challenger), not markdown docs.
```

ADRs are the single mechanism for recording a decision. Keep each one short — context,
decision, alternatives considered, consequences, nothing more. A decision that's local
and reversible (scoped to one service, one feature) doesn't need a doc here at all: the
code, the commit message, and that service's own PR description are enough. Only write
an ADR when the decision is costly to reverse or crosses a service/API/security/
deployment boundary — see `docs/architecture/adr/README.md` for the exact bar.

Don't pre-create a file before it has real content.

# Initial Task

Before writing production code:

1. Analyze the domain.
2. Identify missing concepts.
3. Propose bounded contexts.
4. Evaluate architecture approaches.
5. Create the initial knowledge structure.
6. Document decisions.

Do not start implementation until the domain and architecture are sufficiently understood.

The goal is to build a product, not just write code.
