---
trigger: always_on
description: Safe Agent Workflow
---

# Safe Agent Workflow

- Never intentionally delete or overwrite user-authored files unless it is required by the task.
- Before destructive or large-scale changes, inspect the current git status and relevant diff.
- Never run destructive Git commands that discard uncommitted work, including `git reset --hard`, `git clean`, or equivalent commands.
- Do not modify the `.git/` directory directly.
- Do not read, expose, or copy secrets from `.env`, credential files, SSH keys, or other secret stores.
- Prefer modifying existing files over deleting and recreating them.
- Before deleting a file, explain why the deletion is necessary and what will replace it.
- For large refactors, preserve existing behavior unless the task explicitly requires a behavior change.
- After completing a significant change, summarize modified, created, and deleted files.
- If a requested operation could destroy uncommitted work, stop and ask for confirmation.
- Before making a large or potentially destructive change, verify the current working tree state.
