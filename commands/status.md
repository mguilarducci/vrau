---
description: "Show vrau workflow status"
---

# Vrau Status

Scan `.claude/vrau/workflows/` for folders.

## No Workflows

If directory doesn't exist or is empty:
```
No vrau workflows found.
Use /vrau:start to begin.
```

## List Workflows

For each workflow folder, check which files exist and derive state:

| Files Present | State |
|--------------|-------|
| (empty folder) | Not started |
| brainstorm.md only | Brainstorm complete |
| brainstorm.md + plan.md | Plan complete |
| brainstorm.md + plan.md + execution-log.md | Execution started |

Display:

```
Vrau Workflows:

  2026-01-10-add-auth
    State: Plan complete
    Files: brainstorm.md, plan.md

  2026-01-12-fix-api
    State: Brainstorm complete
    Files: brainstorm.md

  2026-01-13-refactor-db
    State: Not started
    Files: (none)

Commands:
  /vrau:start      - Begin new workflow
  /vrau:brainstorm - Continue brainstorming
  /vrau:plan       - Continue planning
  /vrau:execute    - Continue execution
  /vrau:abort      - Delete workflow
```

## Single Workflow Detail

If user asks about a specific workflow, show more detail:

```
Workflow: 2026-01-10-add-auth

State: Plan complete
Created: 2026-01-10

Files:
  brainstorm.md (2.3 KB)
  plan.md (4.1 KB)

Next step: /vrau:execute
```
