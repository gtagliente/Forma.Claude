# Forma — Requirements, Users, and Open Items (Iteration 1)

_Analyst output. This is a discovery document, not a spec — many items below are questions, not decisions._

## Users (candidate — unconfirmed)

`CLAUDE.md` never explicitly names a user/persona. The Analyst infers at least one persona is required for the described workflows to make sense, but the following are **not confirmed**:

1. **Solo athlete** — manages their own exercises/workouts/routines and logs their own sessions. Strongly implied by "the real execution of a workout by **a user**."
2. **Coach/trainer directing another person** — not mentioned anywhere in `CLAUDE.md`. If Forma needs this, it materially changes the domain model (assignment, visibility, permissions) and should be resolved early, since it affects whether "user" scoping is single-tenant-per-athlete or has a delegation model.
3. **Content curator / library maintainer** — someone (staff? every user?) who defines the canonical Exercise library, if one exists.

## Main workflows (as inferable from the described lifecycle)

1. **Define an exercise** — create/maintain an Exercise (name, instructions, equipment, media, tags, difficulty); optionally trigger/receive AI enrichment.
2. **Compose a workout** — select exercises, define sets/reps/weight/duration/rest/sequencing/supersets/circuits, save as a reusable Workout.
3. **Build a routine** — arrange Workouts across a schedule (e.g. weekly).
4. **Execute a session** — start a Workout Session (presumably from a Routine's scheduled occurrence, or a Workout directly, or possibly ad hoc), record actual reps/weight/duration per set as training happens.
5. **Review progress** — view trends, PRs, volume, consistency derived from session history.

Workflow 4 in particular is under-specified — see open items.

## Missing requirements (not addressed at all in `CLAUDE.md`)

- **Identity & access** — accounts, authentication, authorization, data ownership/privacy. Nothing said.
- **Multi-user relationships** — sharing, following, coach-athlete, social features. Nothing said.
- **Units & localization** — kg vs lb, metric vs imperial, language. Nothing said, but directly affects Workout/Session data shape.
- **Body metrics / goals** — bodyweight, measurements, target goals. Related to Progress Tracking but not mentioned as an input.
- **Notifications/reminders** — nothing said about reminding a user of a scheduled Routine day.
- **Offline / connectivity** — training often happens in gyms with poor connectivity; not addressed.
- **Media handling** — Exercise "media resources" are mentioned but not how they're captured, stored, or sized.
- **Monetization / business model** — entirely absent; may not matter for domain modeling but affects scope.
- **Exercise library governance** — is it curated/shared, user-editable, or both? Directly affects the Exercise bounded-context design.

## Ambiguities in what IS described

- **Workout ↔ Routine reference semantics**: "should reference workouts but should not duplicate workout details" — does a live edit to a Workout retroactively change what a Routine "means," including for Routines already partially executed?
- **Routine scheduling model**: recurring weekly pattern (as shown in the example) vs. specific calendar dates — or both?
- **Session provenance**: must every Workout Session trace back to a specific Workout (and via a Routine occurrence), or can sessions be freestanding ("did an ad hoc session today")?
- **Set-level granularity**: is a "set" a first-class, addressable concept (relevant for comparing planned vs. actual set-by-set, as the example under Workout Session implies), or just a count?
- **Difficulty attribute**: global/objective (set once per Exercise) or subjective per user?
- **Tags**: free-form or controlled vocabulary — matters for search/filtering and for AI enrichment consistency.
- **AI enrichment trigger & trust**: on-demand vs. automatic; does enrichment content require human review/acceptance before it's treated as part of the Exercise's information, or is it always clearly labeled as AI-sourced and separate?

## Recommendation for next iteration

Resolve the **user/persona question** (solo-only vs. coach-athlete) before the Architect commits to bounded contexts involving identity or authorization, since it is the single highest-leverage unknown — it changes the shape of nearly every other concept's ownership model. See `../../scratchpad/open-questions/iteration-1.md`.
