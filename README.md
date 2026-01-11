# Vrau

Workflow orchestration plugin for Claude Code with autonomous execution and user-driven decisions.

## Philosophy

**Autonomous execution, user-driven decisions.**

Vrau automates the development cycle while keeping humans in control of all key decisions:

```
START → BRAINSTORM → REVIEW → PLAN → REVIEW → EXECUTE → DONE
```

## Requirements

- [Claude Code](https://github.com/anthropics/claude-code)
- [Superpowers](https://github.com/obra/superpowers) plugin (required dependency)

## Installation

```bash
claude plugins add mguilarducci/vrau
```

## Commands

| Command | Description |
|---------|-------------|
| `/vrau` | Auto-detect state, continue workflow |
| `/vrau:start` | Begin new workflow |
| `/vrau:status` | Show current state |
| `/vrau:brainstorm` | Continue/refine brainstorm |
| `/vrau:plan` | Continue/refine plan |
| `/vrau:execute` | Continue execution |
| `/vrau:config` | View/edit configuration |
| `/vrau:reset` | Clear state (keeps files) |
| `/vrau:abort` | Abandon workflow |

## Workflow States

1. **idle** - No active workflow
2. **brainstorming** - Creating/refining brainstorm
3. **brainstorm_review** - Awaiting review choice
4. **planning** - Creating implementation plan
5. **plan_review** - Awaiting review choice
6. **executing** - Running the plan
7. **done** - Workflow complete

## Review Options

At each review checkpoint:
- **Guided checklist** - Quick self-review questions
- **Subagent review** - Fresh-eyes analysis with severity ratings
- **Both** - Checklist + subagent in parallel
- **Skip** - Proceed directly

## Configuration

Edit via `/vrau:config` or directly in `.claude/vrau/config.json`:

```json
{
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

## File Structure

```
.claude/vrau/
├── workflows/
│   └── 2026-01-11-feature-name/
│       ├── brainstorm.md         # Committed
│       ├── plan.md               # Committed
│       └── *.local.md            # Local only
├── config.json                   # Committed (team)
├── config.local.json             # Local (personal)
└── state.local.json              # Local (current state)
```

## License

MIT