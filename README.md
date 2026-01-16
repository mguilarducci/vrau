# Vrau

Workflow orchestration plugin for Claude Code.

## Philosophy

Autonomous execution, user-driven decisions.

## Requirements

- [Claude Code](https://github.com/anthropics/claude-code)
- [Superpowers](https://github.com/anthropics/superpowers) plugin

## Installation

```bash
claude plugins add mguilarducci/vrau
```

## Usage

```
/vrau:start
```

Single command handles everything:
- Detects existing workflows and offers to resume
- Creates new workflows with guided setup
- Progresses through phases automatically

## Workflow

```
START → BRAINSTORM → REVIEW → PLAN → REVIEW → EXECUTE → DONE
```

## File Structure

```
docs/designs/<date>-<branch>/
├── README.md       # Workflow metadata
├── design/         # Brainstorm outputs
└── plan/           # Implementation plans
```

## License

MIT
