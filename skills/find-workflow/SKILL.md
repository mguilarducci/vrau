---
name: find-workflow
description: Use when needing to discover, list, or select an existing vrau workflow from .claude/vrau/workflows/. Use before any command that operates on an existing workflow.
---

# Find Workflow

## Overview

Discovers vrau workflows and handles user selection. Returns workflow path, name, state, and files.

## When to Use

- Before operating on an existing workflow
- When user runs /vrau:brainstorm, /vrau:plan, /vrau:execute, /vrau:abort, /vrau:status
- When resuming work after session ends

## Process

1. Scan `.claude/vrau/workflows/` for folders
2. If none exist → display "No workflows found. Use /vrau:start to begin."
3. If one exists → auto-select it
4. If multiple exist → present numbered list with state, ask user to choose

## State Derivation

Derive state from files present in workflow folder:

| Files Present | State |
|---------------|-------|
| (empty folder) | not_started |
| brainstorm.md only | brainstorm_complete |
| brainstorm.md + plan.md | plan_complete |
| + execution-log.md | executing |

## Output Format

After selection, provide:
- **Path:** `.claude/vrau/workflows/<workflow-name>/`
- **Name:** `<workflow-name>`
- **State:** One of the states above
- **Files:** List of files present

## Display Format (for multiple workflows)

```
Found workflows:

1. 2026-01-10-feat-add-auth
   State: plan_complete
   Files: brainstorm.md, plan.md

2. 2026-01-12-fix-api-error
   State: brainstorm_complete
   Files: brainstorm.md

Which workflow? [1-2]
```
