---
description: "Show vrau workflow status"
---

# Vrau Status

## Find Workflows

Scan `.claude/vrau/workflows/` for folders.

If none exist:
```
No vrau workflows found.
Use /vrau:start to begin.
```

## List Workflows

Use the `find-workflow` skill's state derivation for each workflow:

| Files Present | State |
|--------------|-------|
| (empty folder) | Not started |
| brainstorm.md only | Brainstorm complete |
| brainstorm.md + plan.md | Plan complete |
| brainstorm.md + plan.md + execution-log.md | Execution started |

Display:

```
Vrau Workflows:

  <workflow-name>
    State: <state>
    Files: <file list>

Commands:
  /vrau:start      - Begin new workflow
  /vrau:brainstorm - Continue brainstorming
  /vrau:plan       - Continue planning
  /vrau:execute    - Continue execution
  /vrau:abort      - Delete workflow
```

## Single Workflow Detail

If user asks about specific workflow:

```
Workflow: <name>

State: <state>
Created: <date from folder name>

Files:
  <file> (<size>)

Next step: <recommended command>
```
