---
name: workshop-install-all
description: Installs or updates all workshop conventions and subagents into the current project. Writes the delimited conventions block to AGENTS.md and copies subagent .toml files into .codex/agents/. Run this after updating the plugin to a new version.
---

<!-- Generated for Codex — do not edit by hand. -->

# Install / Update All Workshop Assets (Codex)

Installs the latest conventions and all four crew subagents into the current project.

## What gets installed

| Target | Source | Destination |
|--------|--------|-------------|
| Conventions block | `${CODEX_SKILL_DIR}/resources/workshop-conventions.md` | delimited block in `<project>/AGENTS.md` |
| `bricklayer` subagent | `${CODEX_SKILL_DIR}/resources/bricklayer.toml` | `.codex/agents/bricklayer.toml` |
| `plumber` subagent | `${CODEX_SKILL_DIR}/resources/plumber.toml` | `.codex/agents/plumber.toml` |
| `electrician` subagent | `${CODEX_SKILL_DIR}/resources/electrician.toml` | `.codex/agents/electrician.toml` |
| `foreman` subagent | `${CODEX_SKILL_DIR}/resources/foreman.toml` | `.codex/agents/foreman.toml` |

Reminder: there is **no painter subagent**. Painting is a main-thread inline task.

## Steps

1. Create directories if they don't exist:
   - `.codex/agents/`

2. **Install / update conventions in `AGENTS.md`** (same logic as `workshop-install-rules`):
   - Read `${CODEX_SKILL_DIR}/resources/workshop-conventions.md`.
   - If `AGENTS.md` contains a `<!-- BEGIN workshop-conventions -->` / `<!-- END workshop-conventions -->` block, replace its contents (idempotent, no prompt).
   - If `AGENTS.md` exists but contains no such block, ask the user before appending.
   - If `AGENTS.md` does not exist, create it with the delimited block.

3. **Install subagents** — for each file in `${CODEX_SKILL_DIR}/resources/`:
   - `bricklayer.toml`
   - `plumber.toml`
   - `electrician.toml`
   - `foreman.toml`
   - Read the source file.
   - Write to `.codex/agents/{filename}` (overwrite).

4. Show a summary:
   ```
   Workshop assets updated:
     [x] AGENTS.md (workshop-conventions block)
     [x] .codex/agents/bricklayer.toml
     [x] .codex/agents/plumber.toml
     [x] .codex/agents/electrician.toml
     [x] .codex/agents/foreman.toml

   Note: no painter subagent. Paint is handled by the main thread.
   ```

5. Remind the user to commit:
   ```
   git add AGENTS.md .codex/agents/
   git commit -m "Update workshop conventions and subagents"
   ```

6. Tell the user to restart Codex so the new subagents register.
