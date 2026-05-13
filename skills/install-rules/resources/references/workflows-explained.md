# Workflows Explained

A **workflow** is a chain: several agents (and possibly inline steps) running in a defined order to deliver a single outcome. In this plugin, the canonical workflow is `/workshop:build-room`, which builds an entire room end-to-end.

## The orchestrator pattern

One file owns the order. Everything else is a step.

In `/workshop:build-room`, the SKILL.md is the orchestrator. It knows the full topology:

1. `foreman` (agent) → writes `PLAN.md`.
2. `bricklayer` (agent) → builds the walls, drafts `PANEL.md` skeleton.
3. `plumber` (agent) → installs pipes.
4. `electrician` (agent) → edits `PANEL.md` to add circuits.
5. Main thread → paints walls via `/workshop:paint-wall`.

Individual trades stay narrow. `bricklayer` does not know that `plumber` or `electrician` exist. That isolation is what lets you swap one trade without rewriting the others.

## Walking through `/workshop:build-room` line by line

> 1. **Read user request**: which room, where (the site directory).

The orchestrator collects the inputs once and threads them through the workflow. Don't let each step re-ask the user — that's a workflow smell.

> 2. **Plan** — invoke the `foreman` agent.

The plan is written **before** any work happens. This serves two purposes: it documents the intended order, and it gives the user a chance to bail before any artifact is created.

> 3. **Execute the plan**, in order:
>    - Walls (bricklayer)
>    - PANEL.md skeleton (bricklayer, because the electrician can't create files)
>    - Plumbing (plumber, if needed)
>    - Electrical (electrician)

Notice step 3 quietly handles a constraint: `electrician` lacks `Write`, so the orchestrator asks `bricklayer` to scaffold `PANEL.md` first. **The orchestrator is where you compensate for narrow tool scopes.** Each agent stays clean.

> 4. **Finishing paint** — main-thread inline, no agent.

The workflow ends with a step that has no agent. This is the lesson: orchestrators can mix sub-agent calls with inline main-thread work. They are not "agent only" pipelines.

> 5. **Summarise the build** with a table of every artifact produced.

A good orchestrator always ends with a recap. It's the only place that has seen every step, so it's the only place that can produce that recap.

## How `foreman` chains agents

`foreman` is a special case: it is itself an agent that delegates to other agents. When the user asks "build a bathroom" directly (not via `/workshop:build-room`), `foreman` matches and runs its plan + delegate flow.

If the `Task` tool is available, `foreman` actually dispatches sub-agents from inside its own turn. If `Task` is not available in the harness, `foreman` falls back to writing the plan and telling the main thread which agents to invoke next. **A workflow agent should always have a graceful degradation path.**

## Workflow rules of thumb

1. **One file owns the order.** Don't spread orchestration logic across the trades.
2. **Each step produces a tangible artifact.** Debugging a workflow becomes "which step's output is missing?".
3. **The orchestrator decides when to fall back to the main thread.** Some steps are agent-shaped, some aren't. Mix freely.
4. **Re-collect inputs once.** Don't ask the user the same question at every step.
5. **Always end with a summary.** Recap which artifacts exist, who built each, and the overall status.

## Anti-patterns

❌ **The mega-agent.** One agent that does walls + pipes + wiring + paint. Cannot scope tools meaningfully; cannot swap a step; cannot test in isolation.

❌ **Implicit chaining.** Trades that "know" they should call the next trade. Now you can't run a trade standalone — every invocation kicks off everything.

❌ **No plan step.** Jumping straight to delegation without a `PLAN.md`. The plan is the artifact the user reviews before work happens; skipping it removes the bail-out point.

❌ **Orchestrator without a recap.** The user finishes the workflow and doesn't know what was created. Always end with the artifact summary.

## Checklist before adding a workflow

- [ ] Is there a clear order the steps must run in?
- [ ] Does each step produce an inspectable artifact?
- [ ] Is the orchestration logic in exactly one file?
- [ ] Are the individual steps usable standalone too?
- [ ] Does the workflow include a plan step before doing destructive or expensive work?
- [ ] Does it end with a summary?
