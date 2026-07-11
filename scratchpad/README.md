# scratchpad/

## Purpose

Ephemeral, unreviewed working space. This is where an agent thinks out loud, drafts, and critiques — before anything is promoted into the durable `docs/` structure.

## What belongs here

- **Challenger output** — reviews, critiques, and findings about Analyst/Architect proposals (e.g. `challenger-review-iteration-N.md`).
- **`open-questions/`** — unresolved questions that need a future iteration or a human decision before they can be answered. One file per iteration/topic is fine.
- Any other draft/working material an agent wants to keep around without committing it to permanent docs yet.

## What does NOT belong here

- Anything considered settled/durable — once a finding is acted on or a question is answered, the *answer* belongs in `../docs/` (product, architecture, service, or feature scope as appropriate), not here. This folder is expected to accumulate stale material over time; it is not itself promoted wholesale.

## When an agent should load this context

- The Challenger, always — both to write new findings and to check whether past findings were addressed.
- Any agent starting a new iteration, to check `open-questions/` for unresolved items before proposing new work.
- Not authoritative for any agent trying to determine current, agreed state — use `../docs/` for that.
