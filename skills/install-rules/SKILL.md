---
name: install-rules
description: Installs the workshop conventions rule file into the current project's .claude/rules/ directory. Use when the user wants to bring the construction-crew teaching conventions into their project.
---

# Install Workshop Conventions

Read the conventions file at `${CLAUDE_SKILL_DIR}/resources/workshop-conventions.md` and write it to `.claude/rules/workshop-conventions.md` in the current project root.

## Steps

1. Read the file at `${CLAUDE_SKILL_DIR}/resources/workshop-conventions.md`.
2. Check if `.claude/rules/` directory exists in the project. If not, create it.
3. Write the contents to `.claude/rules/workshop-conventions.md`.
4. Confirm to the user that the rule has been installed and remind them to commit:
   ```
   git add .claude/rules/workshop-conventions.md
   git commit -m "Add workshop conventions rule"
   ```

If the file already exists, ask the user whether to overwrite it (it may have been customized).

## Why this skill exists

The conventions file is the teaching crib-sheet for the construction-crew metaphor. Once installed in `.claude/rules/`, Claude reads it on every turn in that project — the lessons about agents vs main-thread, tool scoping, and delegation rules become always-on for any developer working there.
