# Workshop Conventions

These conventions teach how Claude Code agents, skills, and workflows fit together, using a construction-crew metaphor. They are not rules for "real" work — they are the rules of this sandbox, designed to make the patterns visible.

If you internalise the lessons here, you will know:

- When to delegate to a sub-agent and when to do the work on the main thread.
- How agent `description:` fields drive automatic invocation.
- How to scope `tools:` so an agent's identity is enforced at the harness level.
- How to chain agents into workflows without growing them into one giant prompt.

---

## 1. The crew

This plugin installs four agents and zero painter. Memorise the table:

| Agent | Trade | Summon by saying… | Tools |
|---|---|---|---|
| `bricklayer` | Bricklayer | "build a wall", "lay the foundation", "construir un muro" | `Read, Write, Bash` |
| `plumber` | Plumber | "install pipes", "fix the leak", "instalar tubería" | `Read, Write, Bash` |
| `electrician` | Electrician | "wire the circuit", "install a socket", "cablear" | `Read, Edit, Bash` |
| `foreman` | Foreman | "build a whole bathroom", "plan the renovation", "foreman" | `Read, Write, Bash, Task` |
| **(none)** | **Painter** | "paint the wall blue" → **main thread does it** | n/a |

Each agent produces a tangible artifact (`WALL-*.md`, `PIPES-*.md`, `PANEL.md`, `PLAN.md`) inside a chosen site directory (default `./construction-site/`). Inspecting those artifacts is how you see what each layer did.

---

## 2. The painter rule

**Not every task gets an agent.**

When the user says "paint the wall", Claude on the main thread handles it directly via `/workshop:paint-wall`. No sub-agent. The skill body contains the actual procedure inline.

Why? Painting in this sandbox is:

- One step (append a `painted_color:` line).
- Reversible (paint again).
- Has no specialised tooling (the main thread already has `Edit`).
- Has no specialised prompting needs.

A sub-agent here would add latency, indirection, and a second context to reason about — without paying for itself. **Reach for an agent only when the task earns one.**

A task earns an agent when at least one of these is true:

1. It needs a **focused prompt** that would clutter the main thread (e.g. a code reviewer with strict rules).
2. It benefits from a **narrow tool scope** (e.g. read-only investigators, or the `electrician` who lacks `Write`).
3. It is **repeated** across many calls and you want a stable entry point.
4. It produces a **long, side-tracked context** that the main thread doesn't need to carry forward.

If none of those apply, write a main-thread skill.

---

## 3. How agents are summoned

Claude reads the `description:` field of every installed agent on every turn, and matches the user's request semantically. The richer and more specific your description, the more reliably Claude picks the right agent.

**Bad description** (vague — will be missed):

```yaml
description: handles electrical stuff
```

User says "the bulb in the kitchen is out" → Claude probably doesn't make the connection. "Electrical stuff" is too abstract.

**Good description** (rich, with trigger phrases — fires reliably):

```yaml
description: The electrician of the crew. Use whenever the user mentions wiring, circuits, sockets, lights, breakers, or electrical work. Triggers on phrases like "wire the circuit", "install a socket", "the lights don't work", "add a breaker", "cablear", "instalar enchufe", "el panel eléctrico", "no funcionan las luces", "tirar cable". Edits an existing PANEL.md artifact to add circuits. Cannot create new files — only edits.
```

User says "the bulb in the kitchen is out" → "the lights don't work" / "no funcionan las luces" matches → `electrician` fires.

**Rule of thumb:** include 6–10 trigger phrases, in both English and Spanish if your team uses both. State what the agent *does* and what it *cannot do*. Mention the artifacts it produces.

---

## 4. Tool scoping

Every agent declares a `tools:` list. That list is the **enforced contract** between the agent and the harness — it is not just documentation.

Examples in this plugin:

- `bricklayer` has `Write` because a bricklayer creates new structures.
- `plumber` has `Write` for the same reason.
- `electrician` does **not** have `Write`. It has `Edit`. An electrician arrives after the panel skeleton exists and modifies it. If `PANEL.md` is missing, the electrician cannot fake one — it must refuse.
- `foreman` has `Task` because its job is to delegate.

**Why this matters:** prose can ask an agent to "stay in its lane", but tool scoping enforces it. Even if the prompt drifts, the harness will not give the electrician a `Write` call. Treat the tool list as a security boundary, not a hint.

**Principle:** give an agent the minimum tools it needs to do its job. Excess tools encourage scope creep and dilute the agent's identity.

---

## 5. Skills are recipes

A `SKILL.md` is a procedure. Claude reads it when the user invokes the matching slash command (e.g. `/workshop:build-wall`). Skills can:

- Be executed inline on the main thread (e.g. `/workshop:paint-wall`).
- Hand off to a single agent (e.g. `/workshop:build-wall` → `bricklayer`).
- Orchestrate multiple agents (e.g. `/workshop:build-room`).

**Frontmatter** is the minimum contract:

```yaml
---
name: build-wall
description: Build a wall in the chosen construction-site directory. Delegates to the `bricklayer` agent.
---
```

**Body structure** that has served this plugin well:

1. **Why this skill exists** — the lesson it teaches or the value it provides.
2. **What this skill does** — numbered steps.
3. **Do NOT** — explicit anti-patterns.
4. (Optional) **Teaching note** — meta-commentary for the team.

**Bundled resources** live under `${CLAUDE_SKILL_DIR}/resources/`. The install skills in this plugin use that pattern to ship the conventions file and the agent definitions.

**Naming:** slash commands are `/<plugin>:<skill-name>`. Keep skill names short, kebab-case, verb-first.

---

## 6. Workflows are chains

Some tasks need several specialists in a defined order. That is a **workflow**. In this plugin:

- `foreman` (agent) writes a plan.
- `/workshop:build-room` (skill) executes the plan: walls → pipes → wiring → paint.

The orchestrator (whether agent or skill) is the only place that knows the topology. Individual trades stay narrow — `bricklayer` does not know about `plumber`. That isolation is what lets you swap one trade out without rewriting the others.

**Workflow rules of thumb:**

- One file owns the order. Don't spread orchestration logic across the trades themselves.
- Each step produces a tangible artifact. That makes debugging easy: when something goes wrong, you can see which step's output is missing or malformed.
- The orchestrator decides when to fall back to the main thread. In `/workshop:build-room`, painting is the main-thread step at the end.

---

## 7. When to delegate vs not — checklist

**Delegate to an agent when:**

- ✅ The task fits a clear specialty with a name your team would use ("review", "test", "scaffold").
- ✅ You want to scope tools narrowly (read-only investigator, edit-only refactor).
- ✅ The task is repeated across many calls.
- ✅ The task produces a long side-context the main thread doesn't need.

**Do it on the main thread when:**

- ✅ It's a one-off.
- ✅ It needs no specialised tools — the main thread already has them.
- ✅ The whole procedure fits in ten lines of skill body.
- ✅ The work is read-only and trivial (a directory walk, a status report).
- ✅ The work needs the main thread's accumulated context (recent edits, current file).

**Counter-examples that the plugin demonstrates:**

| Want to do this | Right answer | Why |
|---|---|---|
| Build a wall | Agent (`bricklayer`) | Has a named specialty; produces a tangible artifact; repeated. |
| Paint a wall | Main thread | One-line edit; no specialty; reversible. |
| Inspect the site | Main thread | Read-only walk; `Glob` + `Read` suffice. |
| Demolish a file | Main thread | Needs a confirmation gate, not a specialist. |
| Build a room | Workflow (skill + multiple agents) | Multi-trade, ordered, ends in a main-thread paint. |

---

## 8. Artifacts and the site directory

All teaching artifacts live in a directory the user picks at the start of an exercise. Default: `./construction-site/`.

| File | Owner | Purpose |
|---|---|---|
| `WALL-<slug>.md` | `bricklayer` | One wall, with location, dimensions, material, status. |
| `PIPES-<location>.md` | `plumber` | Pipe run with material, pressure, run type. |
| `PANEL.md` | `bricklayer` scaffolds, `electrician` edits | Table of circuits. |
| `PLAN.md` | `foreman` | Ordered work plan for a multi-trade job. |
| `painted_color:` line | `/workshop:paint-wall` (main thread) | Added inline to an existing `WALL-*.md`. |

If you spot agents writing to files they don't own, that's a bug — open the agent's `.md` file and re-read its "What to do when invoked" section.

---

## 9. Cleanup

To uninstall everything this plugin put into a project:

```bash
rm -f .claude/rules/workshop-conventions.md
rm -f .claude/agents/bricklayer.md .claude/agents/plumber.md .claude/agents/electrician.md .claude/agents/foreman.md
# and optionally
rm -rf ./construction-site/
```

Committing the removal:

```bash
git add .claude/
git commit -m "Remove workshop plugin assets"
```

---

## 10. Further reading

The `references/` directory under this plugin's `install-rules/resources/` contains five short docs:

- `agents-explained.md`
- `skills-explained.md`
- `workflows-explained.md`
- `delegation-rules.md`
- `when-not-to-delegate.md`

Read them in that order if you want the long version of any section above.
