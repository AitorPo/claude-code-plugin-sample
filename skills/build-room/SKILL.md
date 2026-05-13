---
name: build-room
description: Build an entire room end-to-end: planning, walls, plumbing, wiring, and finishing paint. Orchestrates the `foreman` agent plus all three trades plus an inline paint step.
---

# Build a Room (orchestrator)

## Why this skill exists

This is the **workflow** skill — it chains agents and a main-thread step into a single deliverable. It teaches three things at once:

1. How a foreman (`foreman`) plans before the trades start.
2. How specialist agents run in a defined order, with each producing a tangible artifact.
3. How the workflow falls back to the main thread for the one step (painting) that has no specialist.

## What this skill does

1. **Read user request**: which room (e.g. "bathroom"), where (the site directory, default `./construction-site/`).
2. **Plan** — invoke the `foreman` agent:

   ```
   foreman, plan a <room> at <site>.
   ```

   The foreman writes `PLAN.md` and returns an ordered list of trades.

3. **Execute the plan**, in order:
   - **Walls**: invoke the `bricklayer` agent to build each wall in the plan. If the plan mentions electrical work, also ask the `bricklayer` to draft an empty `PANEL.md` skeleton (because the `electrician` cannot create it — see `/workshop:wire-circuit`).
   - **Plumbing** (if the room needs it): invoke the `plumber` agent for supply and drain.
   - **Electrical**: invoke the `electrician` agent to add circuits to the existing `PANEL.md`.
4. **Finishing paint** — main-thread inline, no agent:
   - Run `/workshop:paint-wall` (or simply do the equivalent inline edit) for each wall in the room.
   - Color: ask the user, or default to `blanco`.
5. **Summarise the build** with a table of every artifact produced:

   ```
   Room: bathroom
   Artifacts:
     PLAN.md                      (foreman)
     WALL-bathroom-north.md       (bricklayer)
     WALL-bathroom-south.md       (bricklayer)
     PANEL.md                     (bricklayer scaffold, electrician edits)
     PIPES-bathroom.md            (plumber)
     PANEL.md circuits #2, #3     (electrician)
     paint                        (main thread, inline)
   ```

## Do NOT

- Do not skip the `foreman` step even if you "know" the order. The plan file is part of the deliverable — teammates inspect it to learn how trades chain.
- Do not invoke a painter agent. There isn't one. Paint runs inline.
- Do not parallelise the trades. Walls before pipes before wiring before paint. The order matters and is part of the lesson.

## Teaching note

Notice how this skill is the only one that knows about *all* the agents at once. That is intentional — the orchestrator is the place where the topology lives. Individual trade skills stay narrow and ignorant of each other. Compare with `/workshop:build-wall`, which knows only about `bricklayer`.
