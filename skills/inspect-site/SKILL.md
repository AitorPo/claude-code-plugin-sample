---
name: inspect-site
description: Walk through a construction-site directory and produce a status report of every WALL.md, PIPES.md, PANEL.md, and PLAN.md artifact found. Read-only, main-thread only — no agent involved.
---

# Inspect the Site

## Why no agent?

Same reasoning as `/workshop:paint-wall`: this task is **pure read work**. `Glob` + `Read` from the main thread covers it completely. Spinning up an "inspector" agent would be ceremony without benefit.

**Lesson:** read-only, no-specialist tasks belong on the main thread. Agents earn their cost when they bring focused prompting or a custom tool scope — not when they're just doing a directory walk.

## What this skill does (no agent involved)

1. Ask the user which site directory to inspect (default `./construction-site/`).
2. **Glob** the directory for:
   - `WALL-*.md`
   - `PIPES-*.md`
   - `PANEL.md`
   - `PLAN.md`
3. **Read** each file and extract the status / key fields from the frontmatter section:
   - For walls: `location`, `material`, `status`, `painted_color` (if present).
   - For pipes: `location`, `material`, `pressure`, `run_type`, `status`.
   - For the panel: count of circuit rows in the table.
   - For the plan: room and number of steps.
4. **Report** a single table:

   ```
   ## Site report — ./construction-site/

   | Artifact                  | Type   | Status     | Notes                          |
   |---------------------------|--------|------------|--------------------------------|
   | PLAN.md                   | plan   | planned    | bathroom, 5 steps              |
   | WALL-bathroom-north.md    | wall   | built      | ladrillo, painted azul         |
   | WALL-bathroom-south.md    | wall   | built      | ladrillo, unpainted            |
   | PIPES-bathroom.md         | pipes  | installed  | PVC, 3 bar, supply             |
   | PANEL.md                  | panel  | 3 circuits | mains + lighting + sockets     |

   Unpainted walls: 1
   ```

## Do NOT

- Do not modify any file. This is a read-only walk-through. If you find a problem, *report* it; don't fix it.
- Do not invoke any agent. Glob + Read is enough.
- Do not require the user to have a complete build — half-finished sites are valid and inspecting them is useful.

## Teaching note

This skill is a good test case for "would I add an agent here?". The answer is no: there is no specialised knowledge, no narrow tool need, no repeated orchestration. A `glob` and a few `read`s do everything. When in doubt, start with a main-thread skill like this one and only promote to an agent if it grows.
