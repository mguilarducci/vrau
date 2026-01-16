# Vrau

Workflow orchestration plugin for Claude Code. Guides complex tasks through structured phases with review checkpoints.

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

That's it. One command handles everything.

## How It Works

### Starting a New Workflow

1. Run `/vrau:start`
2. Describe what you're working on
3. Choose branch setup (worktree, new branch, or current)
4. Vrau creates a workflow folder and guides you through phases

### Resuming an Existing Workflow

1. Run `/vrau:start`
2. Vrau detects existing workflows and shows options:
   - Resume a workflow
   - Start new
   - Delete old workflow

### The Three Phases

```
┌─────────────┐     ┌──────────┐     ┌─────────────┐
│  BRAINSTORM │ ──▶ │   PLAN   │ ──▶ │   EXECUTE   │
│   + Review  │     │ + Review │     │             │
└─────────────┘     └──────────┘     └─────────────┘
```

**Phase 1: Brainstorm**
- Explore requirements and design
- Ask clarifying questions
- Document decisions
- Review with fresh-eyes analysis

**Phase 2: Plan**
- Break design into tasks
- Define dependencies
- Create implementation plan
- Review for completeness

**Phase 3: Execute**
- Implement task by task
- Commit as you go
- Open PR when done

## Workflow Files

All workflow artifacts are stored in `docs/designs/`:

```
docs/designs/2026-01-16-feat-user-auth/
├── README.md           # Workflow metadata
├── design/
│   └── brainstorm.md   # Phase 1 output
└── plan/
    └── plan.md         # Phase 2 output
```

## State Detection

Vrau knows where you are by checking what files exist:

| Files Present | State | What Happens |
|---------------|-------|--------------|
| README.md only | Not started | Begin brainstorm |
| + design/*.md | Brainstorm done | Begin planning |
| + plan/*.md | Plan done | Begin execution |
| Execution markers | Done | Show summary |

## Philosophy

**Autonomous execution, user-driven decisions.**

Vrau automates the tedious parts while keeping you in control of all key decisions. Each phase has a review checkpoint where you can adjust course.

## License

MIT
