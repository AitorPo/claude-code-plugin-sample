# Agents Explained

An **agent** in Claude Code is a sub-Claude with a focused prompt, a scoped tool list, and a model choice. It runs in a separate context from the main thread and reports back when done.

In our construction crew, each trade (bricklayer, plumber, electrician, foreman) is an agent. Each one knows one job well and ignores everything else.

## Anatomy of an agent file

Every agent lives as a Markdown file in `.claude/agents/`:

```yaml
---
name: bricklayer
description: The bricklayer of the crew. Use whenever the user wants to build a wall, lay a foundation, raise a structure...
tools: Read, Write, Bash
model: sonnet
---

You are the bricklayer of a small construction crew...
```

The three fields that matter most:

### `name`

The string Claude uses to invoke the agent. Match the filename (`bricklayer.md` → `name: bricklayer`). Keep it short and memorable.

### `description` — the most important field

This is what the main-thread Claude reads to decide whether to delegate. It is **the dispatcher**. Vague descriptions cause missed delegations; rich ones make the right agent fire reliably.

**Side-by-side example:**

Bad:

```yaml
description: handles electrical stuff
```

User: "the bulb in the kitchen is out"
→ Main thread: "electrical stuff" is too abstract; I'll just handle it myself.
→ Wrong outcome.

Good:

```yaml
description: The electrician of the crew. Use whenever the user mentions wiring, circuits, sockets, lights, breakers, or electrical work. Triggers on phrases like "wire the circuit", "install a socket", "the lights don't work", "add a breaker", "cablear", "instalar enchufe".
```

User: "the bulb in the kitchen is out"
→ Main thread: "the lights don't work" matches; delegate to `electrician`.
→ Right outcome.

**Pattern that works:**

1. One-line role summary ("The electrician of the crew.")
2. "Use whenever the user mentions …" with the concept-level triggers.
3. "Triggers on phrases like …" with 6–10 concrete example phrases.
4. (Optional) What the agent produces, and what it cannot do.

Multilingual triggers help when your team mixes languages.

### `tools` — the enforced contract

A comma-separated list of tools the agent is allowed to call. This is **not** documentation — the harness actually restricts the agent to these tools.

Example: `electrician` has `Read, Edit, Bash` but no `Write`. If you ask the electrician to create a brand-new `PANEL.md`, it cannot — even if it wanted to. That refusal is the lesson: tool scoping enforces identity at the layer below the prompt.

**Principle of minimum tools:** start with the smallest set that lets the agent do its job and grow only when you have to. Excess tools dilute identity and invite scope creep.

### `model`

Which Claude model the agent runs on. In this plugin every agent uses `sonnet`. Cheaper models (`haiku`) are appropriate for trivial agents; more powerful ones (`opus`) for agents that reason hard. The cost is per-call and adds up — if you have an agent invoked dozens of times in a workflow, model choice matters.

## How dispatch works

On every turn, the main-thread Claude:

1. Reads the user's message.
2. Reads the `description` of every agent in `.claude/agents/`.
3. Decides whether any agent matches well enough to delegate.
4. Either invokes that agent (passing a sub-prompt) or handles the request itself.

If no agent matches, **the main thread does the work**. That fallback is not a failure mode — it's the default. The painter skill in this plugin lives entirely in that default.

## When NOT to make an agent

- **One-off tasks**: a single edit doesn't earn a dedicated agent.
- **Tasks that need the main thread's context**: an agent runs in a fresh context. If your task depends on what just happened on the main thread, an agent loses that.
- **Tasks with no specialty**: if you can't name the specialty in one word, you probably don't have one.

See `when-not-to-delegate.md` for more anti-patterns.

## Checklist before adding a new agent

- [ ] Can I name the specialty in one word? (`reviewer`, `bricklayer`, `tester`)
- [ ] Is the trigger description rich enough that the right requests fire it?
- [ ] Is the tool list the smallest that still works?
- [ ] Will this agent be called more than a handful of times?
- [ ] Is there a reason it shouldn't share the main thread's context?

If you can't tick at least three of those, write a main-thread skill instead.
