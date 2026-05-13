---
name: install-pipes
description: Install pipes in the chosen construction-site directory. Delegates to the `plumber` agent.
---

# Install Pipes

## Why this skill exists

Mirrors `/workshop:build-wall` for plumbing: a thin wrapper that hands the request to the `plumber` agent. The teaching point is that **each trade has its own slash-command entry point**, so the user can invoke the specialist explicitly without having to remember the agent name.

## What this skill does

1. **Parse the request** for:
   - Location (e.g. "kitchen", "bathroom")
   - Material (default `PVC`)
   - Pressure (default `3 bar`)
   - Run type (`supply` or `drain`, default `supply`)
   - Site directory (default `./construction-site/`)
2. **Invoke the `plumber` agent** with a sub-prompt:

   ```
   plumber, please install pipes.
     site:     <site>
     location: <location>
     material: <material>
     pressure: <pressure>
     run_type: <run_type>
   ```

3. **Relay the agent's report back** verbatim.

## Do NOT

- Do not write `PIPES-*.md` yourself.
- Do not paint, wire, or build walls — those are other skills / agents.
- Do not silently turn a "fix the leak" request into a fresh install. Pass the user's exact wording to the agent.

## Teaching note

If you ever find yourself adding logic to this skill that is not "parse + delegate + relay", that logic belongs in the `plumber.md` agent file instead. Skills like this should stay thin.
