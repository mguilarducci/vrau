---
name: find-workflow
description: Use when needing to discover, list, or select an existing vrau workflow from docs/designs/. Use before any command that operates on an existing workflow.
---

# Find Workflow

## Overview

Discovers vrau workflows and handles user selection. Returns workflow path, name, state, and files.

## When to Use

- Before operating on an existing workflow
- When user runs /vrau:brainstorm, /vrau:plan, /vrau:execute, /vrau:abort, /vrau:status
- When resuming work after session ends

## Process

1. Scan `docs/designs/` for folders matching pattern `YYYY-MM-DD-*`
2. If none exist → display "No workflows found. Use /vrau:start to begin."
3. If one exists → auto-select it
4. If multiple exist → present numbered list with state, ask user to choose

## State Derivation

Derive state from folder contents:

| Folder Contents | State |
|-----------------|-------|
| README.md only | not_started |
| README.md + design/*.md | brainstorm_complete |
| README.md + design/*.md + plan/*.md | plan_complete |
| README.md with "Execution complete" | done |

## Output Format

After selection, provide:
- **Path:** `docs/designs/<workflow-name>/`
- **Name:** `<workflow-name>`
- **State:** One of the states above
- **Files:** List of files present
- **Doc Approach:** A (files), B (issues), or C (local-only)
- **No-Commit Mode:** true/false (check for .no-commit.local file)

## Display Format (for multiple workflows)

```
Found workflows:

1. 2026-01-10-feat-add-auth
   State: plan_complete
   Files: design/auth-design.md, plan/auth-plan.md

2. 2026-01-12-fix-api-error
   State: brainstorm_complete
   Files: design/api-error-design.md

Which workflow? [1-2]
```

## No-Commit Mode Detection

Check for `.no-commit.local` file in workflow folder. If present:
- Set `no_commit_mode: true`
- Display warning: "This workflow is in NO-COMMIT mode. Files will NOT be committed."
