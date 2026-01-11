---
description: "View or edit vrau configuration"
---

# Vrau Config

## Load Configuration

1. Read `.claude/vrau/config.json` (team defaults)
2. Read `.claude/vrau/config.local.json` (personal overrides)
3. Merge: local overrides team defaults

## Display Current Config

```
Current configuration:

  Storage:
    workflows: <paths.workflows>

  Reviewer:
    lenses: <reviewer.lenses as comma-separated list>

  Error Handling:
    onBrainstormError: <errorHandling.onBrainstormError>
    onPlanError: <errorHandling.onPlanError>
    onExecutionError: <errorHandling.onExecutionError>
    maxRetries: <errorHandling.maxRetries>

Options:
1. Edit setting
2. Reset to defaults
3. Show config file paths
4. Done
```

## Edit Setting

If user chooses "Edit setting":

```
Which setting to edit?

1. Reviewer lenses
2. onBrainstormError (stop_and_ask | auto_retry_once | log_and_continue)
3. onPlanError (stop_and_ask | auto_retry_once | log_and_continue)
4. onExecutionError (stop_and_ask | auto_retry_once | log_and_continue)
5. maxRetries
6. Cancel
```

Save changes to `.claude/vrau/config.local.json`.

## Default Configuration

```json
{
  "paths": {
    "workflows": ".claude/vrau/workflows"
  },
  "reviewer": {
    "lenses": ["security", "architecture", "testability", "maintainability"]
  },
  "errorHandling": {
    "onBrainstormError": "stop_and_ask",
    "onPlanError": "stop_and_ask",
    "onExecutionError": "auto_retry_once",
    "maxRetries": 1
  }
}
```
