# Skills Explained

A **skill** is a procedure stored as a `SKILL.md` file. Claude reads it when the user invokes the matching slash command (e.g. `/workshop:build-wall`) or when another agent decides to use it. Think of skills as recipes; agents are the cooks.

In this plugin, skills come in three shapes:

1. **Thin wrappers** that hand off to an agent (e.g. `/workshop:build-wall` → `bricklayer`).
2. **Inline procedures** that the main thread executes directly (e.g. `/workshop:paint-wall`).
3. **Orchestrators** that chain several agents and inline steps (e.g. `/workshop:build-room`).

## Anatomy of a SKILL.md

```yaml
---
name: build-wall
description: Build a wall in the chosen construction-site directory. Delegates to the `bricklayer` agent.
---

# Build a Wall

## Why this skill exists
...

## What this skill does
1. ...
2. ...

## Do NOT
...
```

### `name`

Matches the directory name (`skills/build-wall/SKILL.md` → `name: build-wall`). This becomes the slash command suffix.

### `description`

Shown in the slash-command menu and read by Claude when deciding whether to invoke the skill from another context. Keep it specific — "build a wall" beats "constructs structures".

## Body conventions

The teaching-friendly structure used in this plugin:

1. **Why this skill exists** — the lesson or value. This is where new teammates learn what each skill is teaching them.
2. **What this skill does** — numbered steps, in order.
3. **Do NOT** — explicit anti-patterns. (e.g. paint-wall says "do not summon any agent".)
4. (Optional) **Teaching note** — meta-commentary.

Keep skills small. If a skill body grows past one screen, ask whether it's secretly an agent in disguise.

## Bundled resources — `${CLAUDE_SKILL_DIR}/resources/`

Skills can ship files alongside the SKILL.md. The convention is to put them under a `resources/` directory and reference them via `${CLAUDE_SKILL_DIR}/resources/...`.

In this plugin:

- `install-rules/resources/workshop-conventions.md` — the conventions file the install skill writes into `.claude/rules/`.
- `install-agents/resources/{bricklayer,plumber,electrician,foreman}.md` — the four agent definitions the install skill copies into `.claude/agents/`.

When you find yourself wanting to inline a long static file inside a SKILL.md, lift it to `resources/` instead and read it from there. SKILL.md stays small and the resource is reusable.

## Slash-command naming

The full command is `/<plugin>:<skill-name>`. In this plugin's case, `/workshop:build-wall`.

- Lowercase.
- Kebab-case.
- Verb-first when the skill does something (`build-wall`, `install-pipes`, `paint-wall`).
- Noun-first when the skill is a report (`inspect-site`).

Avoid suffixes like `-helper` or `-tool` — they don't add information.

## Skills calling skills

A skill can invoke another skill in its body. `/workshop:build-room` is the canonical example: it walks the trades and ends by invoking `/workshop:paint-wall` for finishing.

Two rules:

1. **Don't form cycles.** Skill A → Skill B → Skill A is a recipe for confusion.
2. **The caller owns the orchestration.** Skills you invoke should not assume they were called from any particular parent — they should still work standalone.

## Skills vs agents — when in doubt

| Question | Skill | Agent |
|---|---|---|
| Is the procedure small (fits on one screen)? | ✅ | maybe |
| Does it need a custom tool scope? | ❌ | ✅ |
| Does it need a focused prompt for a specialty? | ❌ | ✅ |
| Should it run in a fresh context away from the main thread? | ❌ | ✅ |
| Is it invoked by the user via a slash command? | ✅ | ❌ (agents are invoked by Claude) |

Many features need both: a skill for the user-facing entry point, and an agent that the skill hands off to. `/workshop:build-wall` + `bricklayer` is exactly that pairing.

## Checklist before adding a new skill

- [ ] Is the slash-command name short and verb-first?
- [ ] Does the description make it obvious what the skill does?
- [ ] Is the body under one screen? (If not, can a bundled `resources/` file absorb the bulk?)
- [ ] Is there a "Why this skill exists" section so teammates understand the purpose?
- [ ] If it delegates to an agent, does it stay thin (parse + delegate + relay)?
- [ ] If it runs inline, is there a clear reason **not** to use an agent?
