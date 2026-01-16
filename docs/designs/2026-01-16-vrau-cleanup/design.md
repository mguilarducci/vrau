# Vrau Cleanup Design

## Goal

Simplify vrau plugin architecture:
- Single command entry point (`/vrau:start`)
- Consolidate workflow storage to `docs/designs/`
- Remove config system
- File-based state derivation

## Decisions

1. **Single command:** `/vrau:start` handles start AND resume
2. **Workflow location:** `docs/designs/<date>-<branch>/`
3. **State mechanism:** File-based derivation only
4. **Template:** Keep at `.claude/vrau/templates/` as plugin asset

## Final Structure

```
commands/
├── start.md              # Single entry point
└── _phases/
    ├── brainstorm.md
    ├── plan.md
    └── execute.md

skills/
├── find-workflow/        # Discovers workflows
├── vrau/                 # Suggests vrau for complex tasks
├── complexity-calibrated-review/
├── receiving-brainstorm-review/
├── receiving-plan-review/
└── strategic-decomposition/

agents/
└── vrau-reviewer.md

.claude/vrau/
└── templates/
    └── README.template.md
```

## Changes Made

### Deleted

| Path | Reason |
|------|--------|
| `commands/brainstorm.md` | Merged into start.md |
| `commands/plan.md` | Merged into start.md |
| `commands/execute.md` | Merged into start.md |
| `commands/status.md` | Redundant - find-workflow does this |
| `commands/abort.md` | Redundant - just rm -rf |
| `commands/config.md` | Config system removed |
| `.claude/vrau/config.json` | Config system removed |
| `.claude/vrau/workflows/` | Old storage location |

### Updated

| File | Changes |
|------|---------|
| `commands/start.md` | Now handles resume logic |
| `commands/_phases/brainstorm.md` | Paths updated |
| `README.md` | Reflects single command |

## Workflow

```
/vrau:start
    │
    ├── Check for existing workflows (find-workflow skill)
    │   ├── Found → Resume / New / Delete?
    │   └── None → Continue to setup
    │
    ├── Setup (if new)
    │   ├── Get task description
    │   ├── Branch setup
    │   └── Create workflow folder
    │
    └── Phases (automatic progression)
        ├── Brainstorm → Review
        ├── Plan → Review
        └── Execute → Done
```
