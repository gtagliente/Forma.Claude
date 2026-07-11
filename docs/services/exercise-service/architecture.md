# exercise-service — Architecture (Aggregate-Level Guidance)

_Central-loop output (Architect). Guidance at the aggregate-boundary level — not implementation detail (naming, exact field types, migration sequencing are the Service Architect's job, in `Forma.Exercise/docs/architecture/`). Grounded in what's actually in `Forma.Exercise/src` as of this pass, not invented from scratch._

## Aggregate boundary: keep it as one aggregate

`Exercise` is the sole aggregate root for this service; `ExerciseResource` (Media Resource) is correctly a child entity within it, not a separate aggregate — already enforced by an architecture test (`ExerciseDetail_Create_is_called_only_from_Exercise`). Nothing in the new concepts below (ownership, hierarchy, Equipment/Tags/Difficulty) requires splitting this into multiple aggregates; they're all either scalar/value-object additions to `Exercise` or references *out* of it.

## Guidance for adding the not-yet-built concepts (`domain.md`)

- **Ownership/visibility**: an attribute of `Exercise` itself (e.g. an owning `UserId` plus a visibility flag/enum) — stays inside the existing aggregate boundary, no new aggregate needed. References `identity-service`'s `User` by ID only, per ADR-005's independent-datastore rule — never a cross-service join.
- **Exercise hierarchy (parent/child)**: this is Exercise-to-Exercise **across aggregate instances** (a parent `Exercise` and a child `Exercise` are each their own aggregate root/transaction boundary). The child should hold a reference to the parent's ID, never a loaded parent object graph — treat it the same as any other cross-aggregate reference, even though both sides happen to be the same aggregate type. Do not let "it's the same entity type" tempt a design where creating/editing a child cascades into loading and locking the parent aggregate.
- **Equipment/Tags/Difficulty**: likely straightforward additions to `Exercise` (value objects or scalar fields) once the still-open central questions (see `open-questions.md`) resolve whether any of them need independent identity (e.g. Equipment as a referenceable, filterable concept) rather than a free attribute. Don't build ahead of that decision.

## Technical risks observed in the existing codebase (flag, not fix)

The Service Architect should account for these before/while building new features on top — none block starting work, but ignoring them compounds the debt:

1. **Strongly-typed ID migration is in flight but unmerged** (branch `fix/ExerciseResourceMigration_stronglytypedIds`, commit `cc0409d`). Today `Exercise` carries both an inherited `Id` (`Guid`, from `BaseEntity`) and a separate `ExerciseId` property used as the actual EF primary key — a duplicate-identity smell the unmerged branch fixes by making `BaseEntity` generic. Recommend merging that before building the hierarchy self-reference (§ above), since a parent/child `ExerciseId` reference is cleaner to introduce once there's exactly one ID concept, not two.  ==> Fixed
2. **Template-rename debt**: this codebase started as a "Shop/Customer" template; several files/namespaces still say `Shop.Domain.Entities.CustomerAggregate`, `Shop.Query.EventHandlers`, `DeleteCustomerCommandHandler.cs` under the Exercise folder, etc. Not urgent, but worth cleaning up before it misleads someone into thinking "Customer" is a real concept in this service. ==> Fixed
3. **Update/Delete flows for Exercise were never built** — endpoints and command handlers exist only as commented-out stubs. Any new feature that assumes Exercise is editable/deletable (e.g. private-Exercise management, which requires at least Update) needs to actually build this, not assume it exists.
4. **Read model (MongoDB query side) hasn't caught up to `Resources`** — `ExerciseQueryModel` doesn't include Media Resources yet, and the event handler that projects write-side events into the read side only reacts to `ExerciseCreatedEvent`, not Updated/Deleted. New features that need Resources or Update reflected in queries will need this closed first.

## Status

First real pass. Supersedes "placeholder only" in `README.md`.
