---
name: demolish
description: Delete construction-site artifacts (WALL.md, PIPES.md, PANEL.md, PLAN.md). Destructive — always asks for explicit confirmation before removing anything. Main-thread only, no agent.
---

# Demolish

## Why no agent?

Destructive operations don't need a specialist — they need a **confirmation gate**, and that gate is the same regardless of whether an agent or the main thread is acting. Putting demolition behind an agent would only spread the gate logic across two contexts.

**Lesson:** irreversible actions need explicit user confirmation, and that requirement is independent of the agent-vs-main-thread choice. Build the gate at the skill level so it always fires.

## What this skill does (no agent involved)

1. Ask the user **what to demolish**:
   - A single file (e.g. `WALL-bathroom-north.md`)
   - All artifacts of a type (e.g. all `WALL-*.md` in the site)
   - The whole site directory contents (everything)
2. **List exactly which files will be deleted.** Show the full list — never use a glob without expanding it for the user first.
3. **Ask for explicit confirmation** with a fixed phrase. The user must reply with literally `yes, demolish` (or its Spanish equivalent `sí, demoler`). Anything else aborts.
4. **Delete** the listed files with `rm` (or your file-deletion tool). Do not `rm -rf` directories — only delete the files you listed.
5. **Report** which files were removed and which (if any) were spared.

## Confirmation prompt template

```
About to delete the following 3 files:
  ./construction-site/WALL-bathroom-north.md
  ./construction-site/WALL-bathroom-south.md
  ./construction-site/PIPES-bathroom.md

Type "yes, demolish" to proceed. Anything else aborts.
```

## Do NOT

- Do not delete without listing first.
- Do not accept fuzzy confirmations like "ok", "go ahead", "sure". The fixed phrase exists so a careless yes-tap doesn't blow away work.
- Do not delete files outside the site directory, even if the user asks. If they want that, they can use plain shell.
- Do not invoke any agent. The gate logic stays in this skill so it's always visible.

## Teaching note

Compare this skill's structure to `/workshop:paint-wall`. Both are main-thread inline. The difference: paint is trivial and reversible (paint again with a different color), while demolish is destructive. The same architecture (no agent), different ceremony (a confirmation gate). Both choices are deliberate.
