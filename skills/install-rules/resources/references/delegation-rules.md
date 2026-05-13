# Delegation Rules

When should the main thread hand a task to a sub-agent, and when should it just do the work itself? This doc gives concrete rules using the workshop crew as examples.

## The four-question gate

Before delegating, ask:

1. **Is there a clear specialty?** Can I name the trade in one word?
2. **Would tool scoping help?** Does the task benefit from being unable to do X?
3. **Will this be invoked repeatedly?** Or is it a one-off?
4. **Would a fresh context help, or hurt?** Sub-agents lose the main thread's context.

If you get **two or more yeses**, delegate. If not, do the work on the main thread.

## Apply it to each workshop task

### "Build a wall in the garden"

1. Specialty? ✅ bricklayer.
2. Tool scoping? ✅ no `Edit` — walls are written once.
3. Repeated? ✅ yes — every room needs walls.
4. Fresh context? ✅ bricklayer doesn't care about prior conversation.

Four yeses → delegate to `bricklayer`. ✅

### "Paint the wall blue"

1. Specialty? ❌ painter is too thin a specialty for the work it does.
2. Tool scoping? ❌ the main thread already has the right tools (`Edit`).
3. Repeated? ⚠️ yes, but each invocation is one trivial line.
4. Fresh context? ❌ painting often comes right after building — fresh context loses that.

One marginal yes → main thread. ✅

### "Inspect the site"

1. Specialty? ❌ "inspector" is just "directory walk".
2. Tool scoping? ❌ `Glob` + `Read` is the obvious toolset, nothing to scope away.
3. Repeated? ⚠️ yes, but the operation is cheap.
4. Fresh context? ❌ better to share context with the rest of the session.

Main thread. ✅

### "Build a whole bathroom"

1. Specialty? ✅ foreman.
2. Tool scoping? ✅ wants `Task` to delegate.
3. Repeated? ✅ rooms come in groups.
4. Fresh context? ✅ planning is a self-contained chunk.

Delegate to `foreman`. ✅

### "Demolish the bathroom wall"

1. Specialty? ❌ destruction has no expertise — it has a confirmation gate.
2. Tool scoping? ⚠️ you might want to scope away `Write` to enforce read-then-delete, but the gate is the real safety.
3. Repeated? ❌ rare.
4. Fresh context? ❌ destructive ops should stay visible on the main thread, not hidden in a sub-call.

Main thread with a confirmation gate. ✅

## Rules distilled from the examples

**Rule 1: A specialty is something a teammate would name.**

If you can't say "I called the {one word}", you don't have a specialty. "Painter" sounds like one but in this plugin's model it isn't — the work is too thin.

**Rule 2: Tool scoping is a feature, not a side effect.**

`electrician` lacking `Write` is a positive design choice. If you can't articulate which tool you want to exclude and why, you're probably not benefiting from tool scoping.

**Rule 3: Repetition justifies the setup cost.**

Defining an agent has overhead (description, tools, model, prompt). One-time tasks don't pay that back.

**Rule 4: Fresh context cuts both ways.**

A sub-agent is great when you want isolation (e.g. a code reviewer that shouldn't be influenced by what the user just wrote). It's terrible when continuity matters (e.g. paint right after building).

**Rule 5: Destructive ops need gates, not specialists.**

A specialist with a destructive tool is more dangerous than the main thread with a gate. Put the gate at the skill layer.

## When two yeses isn't enough

Sometimes a task ticks two boxes but still belongs on the main thread:

- A read-only sanity check before a destructive op.
- A trivial line edit that follows a long discussion.
- Anything where the user is in a tight back-and-forth and you don't want to break flow.

Use judgement. The four-question gate is a guideline, not a law.

## What about "I'll just make an agent for this rare case"?

Tempting, but think twice. Agents you call once and forget become noise in your `.claude/agents/` directory and dilute the `description:` matching for the agents that matter. Every agent you add competes for Claude's attention.

If you really need a one-off behaviour, write a one-off skill instead. Skills don't compete for attention — they only run when invoked.
