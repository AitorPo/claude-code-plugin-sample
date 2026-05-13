---
name: wire-circuit
description: Wire a new circuit into the PANEL.md of the construction-site directory. Delegates to the `electrician` agent.
---

# Wire a Circuit

## Why this skill exists

Thin wrapper around the `electrician` agent — and a deliberate teaching moment about **tool scoping**. The electrician has `Read + Edit + Bash` but **no `Write`**. That means if `PANEL.md` does not yet exist, the agent will refuse and tell you to call the `bricklayer` first.

If you invoke this skill on a fresh site and it bounces back with "PANEL.md missing", that is not a bug — that is the lesson. Either:

- Ask the `bricklayer` (via `/workshop:build-wall`) to draft an empty `PANEL.md` skeleton, **or**
- Have the main thread scaffold an empty `PANEL.md` and re-invoke this skill.

## What this skill does

1. **Parse the request** for:
   - Room (e.g. "kitchen")
   - Circuit type (`socket`, `lighting`, `appliance`)
   - Amps (default by type: socket=16A, lighting=10A, appliance=25A)
   - Site directory (default `./construction-site/`)
2. **Invoke the `electrician` agent** with a sub-prompt:

   ```
   electrician, please wire a circuit.
     site:  <site>
     room:  <room>
     type:  <type>
     amps:  <amps>
   ```

3. **Relay the agent's report back** verbatim — including any refusal due to missing `PANEL.md`. Do not "fix" the refusal by writing the panel yourself; that would undermine the lesson.

## Do NOT

- Do not create `PANEL.md` from this skill. The whole point is that the electrician lacks Write and so the chain has to be set up correctly.
- Do not edit other artifacts (walls, pipes).

## Teaching note

This is the clearest example in the plugin of why we scope tools narrowly. An electrician with `Write` could draft any file, including pretending to lay walls — which would muddy the agent's role. The minimal toolset enforces the agent's identity at the harness level, not just in prose.
