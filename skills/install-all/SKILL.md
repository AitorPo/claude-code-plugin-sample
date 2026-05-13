---
name: install-all
description: Installs or updates all workshop conventions and agents into the current project in one go. Run this after updating the plugin to a new version.
---

# Install / Update All Workshop Assets

Copies the latest conventions rule and all four crew agents from the plugin into the current project. Overwrites existing files so everything matches the current plugin version.

## What gets installed

| Target | Source | Destination |
|--------|--------|-------------|
| Conventions rule | `install-rules/resources/workshop-conventions.md` | `.claude/rules/workshop-conventions.md` |
| `bricklayer` agent | `install-agents/resources/bricklayer.md` | `.claude/agents/bricklayer.md` |
| `plumber` agent | `install-agents/resources/plumber.md` | `.claude/agents/plumber.md` |
| `electrician` agent | `install-agents/resources/electrician.md` | `.claude/agents/electrician.md` |
| `foreman` agent | `install-agents/resources/foreman.md` | `.claude/agents/foreman.md` |

## Steps

1. Create directories if they don't exist:
   - `.claude/rules/`
   - `.claude/agents/`

2. **Install conventions rule:**
   - Read `${CLAUDE_SKILL_DIR}/../install-rules/resources/workshop-conventions.md`
   - Write to `.claude/rules/workshop-conventions.md` (overwrite)

3. **Install agents** — for each of these files in `${CLAUDE_SKILL_DIR}/../install-agents/resources/`:
   - `bricklayer.md`
   - `plumber.md`
   - `electrician.md`
   - `foreman.md`
   - Read the source file
   - Write to `.claude/agents/{filename}` (overwrite)

4. Show a summary:
   ```
   Workshop assets updated to v1.0.0:
     [x] .claude/rules/workshop-conventions.md
     [x] .claude/agents/bricklayer.md
     [x] .claude/agents/plumber.md
     [x] .claude/agents/electrician.md
     [x] .claude/agents/foreman.md

   Note: there is no painter agent on purpose. Painting is handled inline
   by the main thread via /workshop:paint-wall.
   ```

5. Remind the user to commit:
   ```
   git add .claude/rules/ .claude/agents/
   git commit -m "Update workshop conventions and agents to v1.0.0"
   ```
