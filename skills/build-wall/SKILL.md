---
name: build-wall
description: Build a wall in the chosen construction-site directory. Delegates to the `bricklayer` agent.
---

# Build a Wall

## Why this skill exists

This skill exists to demonstrate the **skill → agent** delegation pattern. The user invokes `/workshop:build-wall`, and this file's only job is to hand the request to the `bricklayer` agent with the right parameters. The skill body does not lay any bricks itself.

The reverse pattern (skill body inline, no agent) is shown in `/workshop:paint-wall`. Compare the two side by side — they are the central teaching contrast of this plugin.

## What this skill does

1. **Parse the user's request** for:
   - Location (where the wall goes)
   - Dimensions (default `3m x 2.5m`)
   - Material (default `ladrillo`)
   - Site directory (default `./construction-site/`)
2. **Invoke the `bricklayer` agent** with a sub-prompt like:

   ```
   bricklayer, please build a wall.
     site:       <site>
     location:   <location>
     dimensions: <w> x <h>
     material:   <material>
   ```

3. **Relay the agent's report back to the user.** Do not summarise — pass through the agent's path + spec verbatim so the user can see exactly what was produced.

## Do NOT

- Do not write `WALL-*.md` yourself. That is the bricklayer's job.
- Do not call `plumber` or `electrician` from this skill — they have their own skills (`/workshop:install-pipes`, `/workshop:wire-circuit`).
- Do not paint after building. The user explicitly invokes `/workshop:paint-wall` if they want that.

## Teaching note

Notice this skill is roughly ten lines of real instructions. That is correct — a skill that wraps a single agent should be small. Its purpose is to give the user a stable slash-command entry point and to standardise the parameters passed to the agent. The agent file (`agents/bricklayer.md`) holds the heavy procedural detail.
