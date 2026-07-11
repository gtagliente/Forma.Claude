# ADR-003: Offline-Capable Workout Session Logging

## Status

Accepted.

## Context

`requirements-and-open-items.md` flagged offline/connectivity as an unaddressed missing requirement: training often happens in gyms with poor or no connectivity, and `CLAUDE.md`'s Workout Session workflow (recording actual reps/weight per set as training happens) is exactly the kind of real-time data entry that connectivity gaps would otherwise block or lose. See `../../../scratchpad/open-questions/iteration-1.md` (#11).

This decision affects Training Execution's consistency model and the overall client/server architecture, so per the Context Promotion Rules it belongs here rather than as a local service decision.

## Decision

**Workout Session logging — and only Workout Session logging — is offline-capable.** While a session is being performed, set-by-set data (reps, weight, duration actually done) is cached locally on the client. Once connectivity is available, the cached session data syncs to the server.

Every other area (Exercise Library, Training Planning) assumes normal connectivity. Browsing/editing Exercises, Workouts, and Routines is not required to work offline.

## Alternatives considered

- **App-wide offline support**: rejected as premature — no described requirement exists for offline Exercise/Workout/Routine editing, and building general-purpose offline sync for read/write plan data is significantly more complex (conflict resolution across concurrently-edited Workout versions, see ADR-002) than the one real, described scenario (logging a session mid-workout).
- **No offline support (assume connectivity)**: rejected — directly contradicts the described training-in-a-gym scenario; losing in-progress set data due to a dropped connection would be a core usability failure for the primary workflow (Workflow 4 in `requirements-and-open-items.md`).

## Consequences

- **Training Execution** (`bounded-contexts.md`, Context 3) must tolerate `WorkoutSession` data arriving from the client after the fact, potentially out of order relative to when it was actually recorded.
- Conflict resolution (e.g. the same session partially synced twice, or a Workout version changing between when a session started offline and when it syncs) and the exact sync protocol/local storage mechanism are implementation details, deferred to the service/technology layer.
- Because a session pins a specific Workout version at start time (ADR-002), a session that starts offline and syncs later still resolves unambiguously to the version that was current when it started — the version pin does not depend on server round-trip timing.

## References

- `../../product/vision.md`
- `../../product/requirements-and-open-items.md` → Missing requirements
- `../bounded-contexts.md` (Context 3)
- `ADR-002-workout-versioning-and-session-snapshot.md`
- `../../../scratchpad/open-questions/iteration-1.md` (#11)
