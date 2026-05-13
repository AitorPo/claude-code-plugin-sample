# Workshop — a construction-crew teaching plugin

A sandbox plugin for learning Claude Code **agents**, **skills**, and **workflows**. Not production: every "build" produces tiny Markdown artifacts (`WALL.md`, `PIPES.md`, `PANEL.md`, `PLAN.md`) so you can see what each layer of the system actually did.

## The Workshop

Imagine Claude as a head contractor. When you ask for a job, Claude either picks a specialist from the crew (an **agent**) or, if no specialist matches, rolls up its sleeves and does it directly. This plugin is a sandbox to learn that mechanic.

There are four specialists in the crew — and **deliberately no painter**. When you say "paint this wall", the main-thread Claude does it itself using a plain skill. That asymmetry is the central lesson of the plugin.

## The crew

| Trade | Agent | What you say to summon them |
|---|---|---|
| Bricklayer | `bricklayer` | "build a wall", "lay foundation", "raise structure", "construir un muro", "levantar pared" |
| Plumber | `plumber` | "install pipes", "fix the leak", "connect water", "instalar tubería", "arreglar fuga" |
| Electrician | `electrician` | "wire the circuit", "install sockets", "the lights don't work", "cablear", "instalar enchufe" |
| Foreman | `foreman` | "build a whole room", "coordinate trades", "what's the plan?", "foreman", "supervisar obra" |
| **Painter** | **none — main thread** | "paint this wall" → Claude does it directly |

## Why no painter?

Not every task needs an agent. Agents cost context and add indirection. Reserve them for:

- (a) work that needs a specific tool set,
- (b) work that benefits from a focused prompt,
- (c) repeatable workflows.

Painting in this sandbox is a one-step skill with no specialised tooling — fall through to the main thread.

If you internalise nothing else from this plugin, internalise this: **recognising which tasks don't need an agent is a senior judgement call**, and the painter omission is here to make that call visible.

## Skills vs agents

| Skill | Who runs it | Why |
|---|---|---|
| `/workshop:build-wall` | invoked → `bricklayer` | needs the bricklayer's procedure |
| `/workshop:install-pipes` | invoked → `plumber` | plumbing knowledge |
| `/workshop:wire-circuit` | invoked → `electrician` | electrical knowledge |
| `/workshop:paint-wall` | main thread directly | trivial, no specialist |
| `/workshop:build-room` | main thread reads, then calls `foreman` | full workflow |
| `/workshop:inspect-site` | main thread directly | read-only walk through |
| `/workshop:demolish` | main thread, asks for confirmation | destructive, gates explicitly |

## Setup — Claude Code

### 1. Generate a marketplace token

Use whatever credentialled Git URL hosts the plugin. (Replace `<YOUR_TOKEN>` and the host below with what your team uses.)

### 2. Add the marketplace

```
/plugin marketplace add https://x-token-auth:<YOUR_TOKEN>@bitbucket.org/your-org/skills.git
```

### 3. Install the plugin

```
/plugin install workshop@workshop
```

### 4. Install rules and agents (once per project, re-run after plugin updates)

```
/workshop:install-all
```

Then commit:

```bash
git add .claude/rules/ .claude/agents/
git commit -m "Add workshop conventions and agents"
```

## Setup — Codex

Once the `workshop` bundle is installed in Codex, run inside your project:

```
run workshop-install-all
```

Then commit:

```bash
git add AGENTS.md .codex/agents/
git commit -m "Add workshop conventions and subagents"
```

## Available skills

| Skill | Description | Usage |
|---|---|---|
| `install-rules` | Install workshop conventions rule | `/workshop:install-rules` |
| `install-agents` | Install the four crew agents | `/workshop:install-agents` |
| `install-all` | Install/update all rules + agents | `/workshop:install-all` |
| `build-wall` | Build a wall via `bricklayer` | `/workshop:build-wall` |
| `install-pipes` | Run pipes via `plumber` | `/workshop:install-pipes` |
| `wire-circuit` | Wire a circuit via `electrician` | `/workshop:wire-circuit` |
| `paint-wall` | Paint a wall **on the main thread, no agent** | `/workshop:paint-wall` |
| `build-room` | Full multi-trade workflow + finishing paint | `/workshop:build-room` |
| `inspect-site` | Read-only site status report | `/workshop:inspect-site` |
| `demolish` | Delete artifacts with explicit confirmation | `/workshop:demolish` |

## Available agents

| Agent | Description | Triggers on |
|---|---|---|
| `bricklayer` | Bricklayer — builds walls | "build a wall", "construir un muro", "lay foundation" |
| `plumber` | Plumber — pipes and leaks | "install pipes", "arreglar fuga", "connect water" |
| `electrician` | Electrician — edits `PANEL.md` only | "wire the circuit", "the lights don't work", "cablear" |
| `foreman` | Foreman — plans and delegates | "build a whole room", "supervisar obra", "what's the plan?" |

## Exercises

Work through these in order. Each one isolates one teaching point.

1. **Ask "build a wall in the garden"**. Observe `bricklayer` getting called. Open `agents/bricklayer.md` and find the `description:` field that matched. Notice the list of trigger phrases — that's what made the dispatch work.

2. **Ask "paint the kitchen wall blue"**. Observe the main thread handling it directly, without any agent. Compare agent descriptions side by side: none mention paint. The painter doesn't exist. The main thread is the fallback, and the fallback is fine.

3. **Run `/workshop:build-wall` explicitly**. Watch the skill call out to the `bricklayer` agent. Read `skills/build-wall/SKILL.md` and notice how thin it is — it just parses + delegates + relays. That thinness is intentional.

4. **Run `/workshop:paint-wall`**. Watch the main thread execute the skill body inline. No sub-agent spins up. Compare its `SKILL.md` to `build-wall/SKILL.md` — the asymmetry is the lesson.

5. **Ask "build a whole bathroom"**. `foreman` plans first, writing `PLAN.md`, then delegates `bricklayer` → `plumber` → `electrician` in sequence. Notice that painting is the final step and runs on the main thread, not via a fifth agent.

6. **Modify `agents/electrician.md` to remove its trigger keywords** (gut the `description:` field, leave only "the electrician"). Reinstall with `/workshop:install-agents`. Ask "wire the lamp" again — what happens now? The agent stops firing because the description no longer matches. Restore the full description and re-run the exercise to confirm it fires again.

7. **Add a new agent file `agents/painter.md`** with a description in English like "paint walls in the chosen color, triggers on 'paint the wall', 'pinta la pared'". Reinstall agents. Ask "paint the wall blue" — observe that the new agent now activates. Then *delete* it again and discuss with the team: was adding it actually an improvement, or did it add latency for no gain? (The plugin's argument is the latter.)

8. **Inspect the `tools:` line in each agent file**. Explain why `plumber` has `Write` but `electrician` doesn't. Why does `bricklayer` not have `Edit`? Why does `foreman` need `Task`? The answers are in `agents-explained.md` and in the agent files themselves — but you should be able to articulate them aloud before checking.

## Cleanup

To uninstall everything this plugin put into a project:

```bash
rm -f .claude/rules/workshop-conventions.md
rm -f .claude/agents/bricklayer.md .claude/agents/plumber.md .claude/agents/electrician.md .claude/agents/foreman.md
rm -rf ./construction-site/   # optional: the artifact directory
```

Then:

```bash
git add .claude/
git commit -m "Remove workshop plugin assets"
```

## Further reading

Inside `skills/install-rules/resources/references/`:

- `agents-explained.md` — what `description:` and `tools:` really do.
- `skills-explained.md` — `SKILL.md` anatomy, bundled resources, naming.
- `workflows-explained.md` — orchestration patterns; `build-room` line by line.
- `delegation-rules.md` — concrete rules and counter-examples.
- `when-not-to-delegate.md` — anti-patterns to avoid.

## Auto-configuration

Add to your project's `.claude/settings.json` so new developers get the marketplace automatically:

```json
{
  "extraKnownMarketplaces": ["https://x-token-auth:<YOUR_TOKEN>@bitbucket.org/your-org/skills.git"]
}
```
