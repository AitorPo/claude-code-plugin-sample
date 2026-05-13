---
name: workshop-install-agents
description: Installs the four workshop construction-crew subagents (bricklayer, plumber, electrician, foreman) into the current project's .codex/agents/. Note there is no painter subagent on purpose — painting is handled inline by the main thread.
---

<!-- Generated for Codex — do not edit by hand. -->

# Install Workshop Subagents (Codex)

Copies the four crew subagent definitions into the current project.

## What gets installed

| Subagent | Source | Destination |
|---|---|---|
| `bricklayer` | `${CODEX_SKILL_DIR}/resources/bricklayer.toml` | `.codex/agents/bricklayer.toml` |
| `plumber` | `${CODEX_SKILL_DIR}/resources/plumber.toml` | `.codex/agents/plumber.toml` |
| `electrician` | `${CODEX_SKILL_DIR}/resources/electrician.toml` | `.codex/agents/electrician.toml` |
| `foreman` | `${CODEX_SKILL_DIR}/resources/foreman.toml` | `.codex/agents/foreman.toml` |

There is **no painter subagent**. That is intentional — painting is a main-thread inline skill (`workshop-paint-wall` if you choose to port it). Do not invent one.

## Steps

1. Create `.codex/agents/` if it does not exist.
2. For each of the four files in `${CODEX_SKILL_DIR}/resources/`:
   - Read the source.
   - Write to `.codex/agents/{filename}` (overwrite).
3. Show a summary and remind the user to commit:
   ```
   git add .codex/agents/
   git commit -m "Add workshop construction-crew subagents"
   ```
4. Tell the user to restart Codex so the new subagents register.
