---
name: workshop-install-rules
description: Installs the workshop (construction-crew teaching) conventions into the project's AGENTS.md as a delimited, idempotent block. Invoke this skill inside your project.
---

<!-- Generated for Codex — do not edit by hand. -->

# Install Workshop Conventions (Codex)

Append or update a delimited `workshop-conventions` block in the project's `AGENTS.md` with the latest workshop conventions. Codex auto-loads `AGENTS.md`, so this makes the teaching conventions always-on for any session working in that project.

## Source

Read the conventions file at `${CODEX_SKILL_DIR}/resources/workshop-conventions.md`.

## Steps

1. Determine the target path: `<project-root>/AGENTS.md`.

2. Check the current state of `AGENTS.md`:
   - **If it does not exist:** create it, then go to step 4 (write the block as the file's only content).
   - **If it exists and contains a block delimited by**
     `<!-- BEGIN workshop-conventions -->` **and** `<!-- END workshop-conventions -->`**:**
     replace the contents of that block with the current conventions. **No prompt** — the delimiters mark this as managed content, so re-running is idempotent.
   - **If it exists but contains no such block:** warn the user that you will append to their existing `AGENTS.md` and ask for explicit confirmation before proceeding. Do not overwrite surrounding content.

3. Read `${CODEX_SKILL_DIR}/resources/workshop-conventions.md` into memory.

4. Write the block in this exact shape (with a blank line separator from any preceding content):

   ```markdown

   <!-- BEGIN workshop-conventions -->
   <!-- Managed by workshop-install-rules skill — do not edit by hand. -->

   <contents of workshop-conventions.md>

   <!-- END workshop-conventions -->
   ```

5. Report what happened: "Created / Updated / Appended `workshop-conventions` block in `AGENTS.md`."

6. Remind the user to commit:
   ```
   git add AGENTS.md
   git commit -m "Add workshop conventions to AGENTS.md"
   ```

## Uninstall

Delete everything from the `<!-- BEGIN workshop-conventions -->` line through the `<!-- END workshop-conventions -->` line (inclusive). No other file modifications are needed.
