---
name: paint-wall
description: Paint a wall by appending a `painted_color` line to its WALL.md artifact. Runs inline on the main thread — does NOT delegate to any agent.
---

# Paint a Wall

## Why no agent?

This is the **contrast skill** of the plugin. Compare its body to `/workshop:build-wall`:

- `/workshop:build-wall` is a thin wrapper that hands off to the `bricklayer` agent.
- `/workshop:paint-wall` does the work **inline on the main thread**. There is no painter agent.

Painting in this teaching model is a single-step, no-specialist task. The main-thread Claude already has `Edit` and `Read`. Adding a "painter" agent would only add:

- An extra layer of indirection (skill → agent → tool).
- A second context the user has to reason about.
- Latency, since spinning up a sub-agent costs roundtrips.

**Lesson:** prefer inline skills for trivial tasks. Reach for an agent only when you need specialised prompting, a custom tool scope, or repeatable orchestration.

## What this skill does (no agent involved)

1. Ask the user **which wall** to paint (path to a `WALL-*.md` file, or pick one from the site directory) and **which color**.
2. **Locate** the target file with `ls`/`Read`. If multiple `WALL-*.md` exist and the user didn't specify, list them and ask.
3. **Append** a single line to the file's frontmatter section:

   ```
   painted_color: <color>
   ```

   If a `painted_color:` line already exists, replace it (repainting is allowed).

4. **Report back** with the file path and the new color:

   ```
   Painted ./construction-site/WALL-garden.md
     painted_color: azul
   ```

## Do NOT

- Do not summon any agent. There is no painter agent and there should not be one. If you find yourself wanting one, you are over-engineering — re-read the "Why no agent?" section above.
- Do not touch `PIPES.md`, `PANEL.md`, or `PLAN.md` — they are not walls.
- Do not invent new wall files. Painting only modifies existing artifacts.

## Teaching note

If a teammate asks "why doesn't this skill use a sub-agent like the others?", point them here. The asymmetry is the lesson: **not every skill needs an agent, and recognising which ones don't is a senior-developer judgement call** that this plugin is trying to build.
