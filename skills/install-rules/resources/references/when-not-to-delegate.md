# When NOT to Delegate

The complement to `delegation-rules.md`. This doc catalogues the anti-patterns this plugin is built to teach against. If you find yourself drifting toward any of these, stop and re-read.

## Anti-pattern 1: An agent per file type

> "Let's have a `vue-agent`, a `js-agent`, a `css-agent`, a `markdown-agent`…"

Tempting and wrong. File types are not specialties. The actual specialty is the *intent* (review, scaffold, refactor, test), and one agent of each intent can handle several file types.

In our crew, the trades are by job (build, plumb, wire) not by material (brick-agent, wood-agent, copper-agent, PVC-agent). The latter would multiply agents without adding clarity.

## Anti-pattern 2: An agent for every skill

> "Every skill should call an agent — that's clean architecture."

No. `/workshop:paint-wall`, `/workshop:inspect-site`, and `/workshop:demolish` are all main-thread inline. They earn no agent because:

- The procedure is short.
- The tools the main thread already has cover the work.
- A sub-agent would add latency and indirection.

**A skill is allowed to be its own implementation.** Don't add an agent just to "look architectural".

## Anti-pattern 3: A painter agent because the metaphor demands one

This is the lesson the plugin is named after. The construction metaphor suggests a painter as the obvious next role, but we deliberately don't have one. The reason: in this sandbox, painting is too thin to earn an agent.

If you find yourself adding agents to round out a metaphor rather than to do real work, you are over-fitting. Cut them.

## Anti-pattern 4: Trivial edits behind an agent

> "Let me spin up an agent to append a line to this file."

A sub-agent costs roundtrips and context. For a one-line edit the main thread can do in milliseconds, an agent is pure overhead.

Threshold: if the procedure fits in 3–5 numbered steps and uses tools the main thread has, do it inline.

## Anti-pattern 5: Confirmation gates inside an agent

> "Demolition is dangerous — let's have a `demolisher` agent that asks for confirmation."

Two problems:

1. The user has to trust *both* the main thread and the agent. Two layers, two places the gate can fail.
2. Confirmation is structural, not specialist. There's nothing about destruction that an agent does better than the main thread.

Put the gate at the skill layer (see `/workshop:demolish`). Same gate, fewer moving parts.

## Anti-pattern 6: Implicit chains between agents

> "When `bricklayer` finishes, it should automatically call `plumber`."

No. Agents should not know about each other. The orchestrator owns the chain. If you bake chaining into the trades, you can no longer:

- Run a trade standalone.
- Swap a trade for a different implementation.
- Reorder steps in a new workflow.

The foreman exists precisely so the trades don't have to know each other.

## Anti-pattern 7: Mega-agents that "do everything"

> "Let's have a `house-builder` agent that builds walls, runs pipes, wires circuits, and paints."

What you get:

- A tool list that's the union of every trade's tools — `Read, Write, Edit, Bash, Task` — which is no scoping at all.
- A description that has to match every kind of construction request, competing with itself.
- A prompt that's a thousand lines long.

Splitting into trades + an orchestrator is more code in count but less complexity per piece.

## Anti-pattern 8: Read-only investigations behind an agent

> "Let's have an `inspector` agent for walking the site."

`Glob` + `Read` from the main thread does this in seconds. The main thread is already in the project's context. Spinning up a sub-agent for this is a textbook case of premature delegation.

`/workshop:inspect-site` is deliberately main-thread to drive the point home.

## Anti-pattern 9: Sub-agents that need the main thread's context

> "The agent will figure out the recent conversation."

It won't — by design. Sub-agents run in fresh contexts. If your task depends on "what we were just discussing", an agent will lose that. Stay on the main thread.

## Anti-pattern 10: Agents you'll only call once

If an agent is going to be invoked once and never again, the setup cost (defining it, installing it, maintaining its description so it doesn't false-fire) isn't paid back. Use a one-off skill or just inline the work.

---

## The summary version

Don't delegate when:

- The procedure is short.
- The main thread has all the tools needed.
- The task is destructive (use a gate at the skill layer instead).
- The task is read-only and trivial.
- You're choosing the agent to satisfy a metaphor rather than a need.
- The task needs continuity with what just happened on the main thread.
- The agent would only run once.
- The agent would essentially have every tool — that's not scoping.

When in doubt, start with a main-thread skill. You can always promote it to an agent later if the case grows.
