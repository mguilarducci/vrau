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
3. Choose tracking mode:
   - **GitHub Issue** - Track via issue comments, PRs for reviews
   - **None** - File-based state detection only
4. Choose branch setup (worktree or new branch)
5. Vrau creates a workflow folder and guides you through phases

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
│   + Review  │     │ + Review │     │   + PR      │
└─────────────┘     └──────────┘     └─────────────┘
```

---

### Phase 1: Brainstorm

```
┌──────────────────────────────────────────────────────────────────┐
│  STEP 1: BRAINSTORMING (sonnet)                                  │
│  Invoke superpowers:brainstorming skill                          │
│  → Clarifying questions → Design exploration → Document          │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│  STEP 2: SAVE & CREATE PR                                        │
│  Save to design/brainstorm.md, run /commit-push-pr               │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│  STEP 3: PR-BASED REVIEW LOOP                                    │
│  Spawn reviewer → /review-comment                                │
│  → APPROVED: /merge-pr                                           │
│  → REVISE: /read-review-update-pr, loop (max 3x)                 │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
                        [PHASE 2: PLAN]
```

---

### Phase 2: Plan

```
┌──────────────────────────────────────────────────────────────────┐
│  WRITE PLAN (opus)                                               │
│  Invoke superpowers:writing-plans skill                          │
│  → Task breakdown → Dependencies → Implementation steps          │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│  PR-BASED REVIEW LOOP                                            │
│  Run /commit-push-pr, spawn reviewer → /review-comment           │
│  → APPROVED: /merge-pr                                           │
│  → REVISE: /read-review-update-pr, loop (max 3x)                 │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
                        [PHASE 3: EXECUTE]
```

---

### Phase 3: Execute

```
┌──────────────────────────────────────────────────────────────────┐
│  EXECUTION (model per task from plan)                            │
│  → Parse dependency graph → Dispatch parallel groups             │
│  → Code review after each group                                  │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│  FINALIZE                                                        │
│  Verification → Final PR (closes issue if tracking)              │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
                        [WORKFLOW COMPLETE]
```

## Slash Commands

Vrau provides atomic operations as slash commands:

| Command | Purpose |
|---------|---------|
| `/start-job` | Verify/create GitHub issue, link to workflow |
| `/track-task` | Post status update comment to linked issue |
| `/commit-push-pr` | Stage, commit, push, create PR atomically |
| `/review-comment` | Post structured review as PR comment |
| `/read-review-update-pr` | Read review, make changes, push |
| `/merge-pr` | Merge approved PR to main |
| `/sync-main` | Fetch and merge main into current branch |

## Workflow Files

All workflow artifacts are stored in `docs/designs/`:

```
docs/designs/2026-01-16-feat-user-auth/
├── README.md           # Workflow metadata (tracking mode, issue link)
├── design/
│   └── brainstorm.md   # Phase 1 output
└── plan/
    └── plan.md         # Phase 2 output
```

## Enhanced Plan Format

Vrau plans include dependency tracking for parallel execution:

### Task Dependencies

Each task specifies what it depends on:

```markdown
### Task 3: Feature Implementation

**Depends on:** Task 1, Task 2
**Parallel group:** B
**Model:** sonnet
```

### Parallel Execution Groups

Tasks in the same parallel group run concurrently:

| Group | Tasks | Runs After |
|-------|-------|------------|
| A | 0, 1 | (start) |
| B | 2, 3, 4 | Group A |
| C | 5 | Group B |

### Model Per Task

Each task specifies which model to use:
- **haiku**: Simple setup, config
- **sonnet**: Standard implementation
- **opus**: Complex logic (requires approval)

## State Detection

Vrau knows where you are by checking workflow contents:

**Tracking Mode: GitHub**

Uses issue comments for status. PRs track review progress.

**Tracking Mode: None**

| Files Present | State | What Happens |
|---------------|-------|--------------|
| README.md only | Not started | Begin brainstorm |
| + design/*.md | Brainstorm done | Begin planning |
| + plan/*.md | Plan done | Begin execution |

When tracking mode is None, vrau asks user to confirm detected state before proceeding.

## Philosophy

**Autonomous execution, user-driven decisions.**

Vrau automates the tedious parts while keeping you in control of all key decisions. Each phase has a PR-based review checkpoint where you can adjust course.

## License

MIT
