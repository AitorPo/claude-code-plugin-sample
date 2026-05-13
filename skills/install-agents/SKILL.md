---
name: install-agents
description: Installs the four workshop agents (bricklayer, plumber, electrician, foreman) into the current project's .claude/agents/ directory. Use when the user wants to register the construction-crew specialists in their project.
---

# Install Workshop Agents

Copy the four crew agent files from `${CLAUDE_SKILL_DIR}/resources/` to `.claude/agents/` in the current project.

## What gets installed

| Agent file | Destination |
|---|---|
| `bricklayer.md` | `.claude/agents/bricklayer.md` |
| `plumber.md` | `.claude/agents/plumber.md` |
| `electrician.md` | `.claude/agents/electrician.md` |
| `foreman.md` | `.claude/agents/foreman.md` |

Note there is **no painter agent**. That is intentional — painting is a main-thread skill (`/workshop:paint-wall`). Do not invent one.

## Steps

1. Create `.claude/agents/` if it does not exist.
2. For each of the four files in `${CLAUDE_SKILL_DIR}/resources/`:
   - Read the source.
   - Write to `.claude/agents/{filename}` (overwrite).
3. Show a summary and remind the user to commit:
   ```
   git add .claude/agents/
   git commit -m "Add workshop construction-crew agents"
   ```

## Why this skill exists

Agents only become discoverable to Claude once their `.md` file is in `.claude/agents/`. This skill is the standard "register the crew" step — run it once per project, then again after each plugin update.
