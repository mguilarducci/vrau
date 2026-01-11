# Vrau Plugin - Design Document

> **For Implementation:** Use `superpowers:writing-plans` to create a detailed implementation plan from this design. This document is self-contained and requires no additional context.

---

## Overview

**Vrau** is a workflow orchestration plugin for Claude Code that automates the development cycle while keeping humans in control of all decisions.

**Philosophy:** Autonomous execution, user-driven decisions.

**Workflow:**
```
START → BRAINSTORM → REVIEW → PLAN → REVIEW → EXECUTE → DONE
```

**Key Characteristics:**
- Distributed as a plugin via marketplace
- Requires Superpowers as a dependency
- Project-level storage in `.claude/vrau/`
- Explicit state tracking via JSON file
- 9 user-invoked commands for full control
- Model-invoked skill suggests vrau for complex tasks

---

## Integration with Superpowers

Vrau orchestrates the flow; Superpowers does the heavy lifting.

| Vrau Phase | Superpowers Skill | When Used |
|------------|-------------------|-----------|
| Start (worktree option) | `superpowers:using-git-worktrees` | User chooses "Create worktree" |
| Brainstorm | `superpowers:brainstorming` | Always |
| Plan | `superpowers:writing-plans` | Always |
| Execute | `superpowers:executing-plans` | User chooses "Parallel session" |
| Execute | `superpowers:subagent-driven-development` | User chooses "Subagent-driven" |
| Finish | `superpowers:finishing-a-development-branch` | After execution completes |
| Review (subagent) | `superpowers:requesting-code-review` pattern | User chooses subagent review |

**Other Superpowers invoked indirectly:**
- `superpowers:verification-before-completion` - During execution
- `superpowers:receiving-code-review` - When handling feedback
- `superpowers:test-driven-development` - If plan includes TDD tasks
- `superpowers:systematic-debugging` - If execution encounters bugs

---

## Workflow States

**7 States:**

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│   IDLE ──► BRAINSTORMING ──► BRAINSTORM_REVIEW ──► PLANNING        │
│     ▲                              │                    │           │
│     │                              │ (skip/approve)     ▼           │
│     │                              │            PLAN_REVIEW         │
│     │                              │                    │           │
│     │                              │ (skip/approve)     ▼           │
│     │                              └──────────────► EXECUTING       │
│     │                                                   │           │
│     │                                                   ▼           │
│     └─────────────────────────────────────────────── DONE          │
│                     (cleanup: keep/archive/delete)                  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

| State | Description | Next Actions |
|-------|-------------|--------------|
| `idle` | No active workflow | `/vrau:start` |
| `brainstorming` | Creating/refining brainstorm | Continue or complete |
| `brainstorm_review` | Brainstorm done, awaiting review choice | Checklist / Subagent / Both / Skip |
| `planning` | Creating implementation plan | Continue or complete |
| `plan_review` | Plan done, awaiting review choice | Checklist / Subagent / Both / Skip |
| `executing` | Running the plan | Autonomous until done or error |
| `done` | Workflow complete | Keep / Archive / Delete |

**Transitions:**
- User decisions drive state transitions (not automatic)
- `/vrau:abort` can exit from any state → `idle`
- `/vrau:reset` clears state → `idle` (keeps files)

---

## Start Flow

**`/vrau:start` presents 3 steps:**

### Step 1: Task Description
```
What are you working on?
> [User describes task in one sentence]
```

### Step 2: Branch Name
```
Suggested branch: fix/payment-api-timeout

1. Confirm
2. Edit name

Choose:
```

**Validation:** Branch names must match `^[a-zA-Z0-9/_-]+$`

### Step 3: Workspace Setup
```
How do you want to work?

1. Create worktree (isolated workspace)
   → Uses superpowers:using-git-worktrees
   → New directory with separate branch
   → Best for: larger features, experiments, parallel work

2. Work in current branch
   → Stay in current directory, no branch change
   → Best for: small fixes, quick changes

3. Create new branch (no worktree)
   → New branch in current directory (auto checkout)
   → Best for: medium tasks, standard git flow

Choose:
```

### Step 4: Confirmation
```
✓ Branch created: fix/payment-api-timeout
✓ Workflow initialized
✓ State: brainstorming

Starting brainstorm phase...
```

---

## Review System

**Review Trigger:** After brainstorm or plan completes, vrau presents:

```
[Brainstorm/Plan] complete!

How would you like to review?

1. Guided checklist
   → Quick self-review with key questions
   → You check items, then approve/revise

2. Subagent review
   → Fresh-eyes analysis by reviewer agent
   → Returns Critical/Important/Minor feedback

3. Both in parallel
   → Checklist shown now
   → Subagent runs in background
   → Combined feedback for decision

4. Skip review
   → Proceed directly to next phase

Choose:
```

### Guided Checklists (context-aware)

**Brainstorm Checklist:**
- ☐ Is the scope clear?
- ☐ Are requirements explicit?
- ☐ Are alternatives considered?
- ☐ Are dependencies identified?
- ☐ Is it the simplest solution?

**Plan Checklist:**
- ☐ Are file paths exact?
- ☐ Are steps bite-sized (2-5 min)?
- ☐ Is code complete (no placeholders)?
- ☐ Are commands with expected output?
- ☐ Is TDD included where needed?

### Subagent Reviewer

**Characteristics:**
- Generic agent with configurable lenses
- Fresh context (no bias from main conversation)
- Structured output: Summary → Critical → Important → Suggestions → Verdict

**Default Lenses (4):**
1. Security
2. Architecture
3. Testability
4. Maintainability

**Additional Lenses (user-configurable):**
- Performance
- UX
- Observability
- Integrations

### After Review
```
Review complete.

1. Approve - Proceed to next phase
2. Revise - Update based on feedback
3. View full review

Choose:
```

---

## Execution Phase

**Execution Options:**
```
Plan approved! Ready to execute.

How do you want to run the plan?

1. Subagent-driven (this session)
   → Uses superpowers:subagent-driven-development
   → Fresh subagent per task
   → Autonomous execution
   → Progress tracked in execution-log.local.md

2. Parallel session (separate terminal)
   → Uses superpowers:executing-plans
   → Open new terminal in worktree/branch
   → Batch execution with checkpoints
   → Return here when done

Choose:
```

**Autonomous Execution Behavior:**
- Runs with minimal human intervention
- Only stops for:
  - Errors requiring decision (based on config)
  - Completion
- No "task done, proceed?" prompts between tasks

**Execution Log Format (`execution-log.local.md`):**
```markdown
# Execution Log: fix-payment-api-timeout
Started: 2026-01-11 10:30:00

## Task 1: Create timeout config
Status: ✓ Complete
Duration: 3m 42s
Commit: a1b2c3d

## Task 2: Update API client
Status: ✓ Complete
Duration: 5m 18s
Commit: e4f5g6h

## Task 3: Add retry logic
Status: ⚠ Error - retrying
Error: Test failed - connection mock missing
Retry: 1/1
Status: ✓ Complete
Duration: 4m 55s
Commit: i7j8k9l

## Summary
Total tasks: 3
Completed: 3
Duration: 13m 55s
```

---

## File Structure

**Project Structure:**
```
.claude/vrau/
├── workflows/
│   └── 2026-01-11-fix-payment-api-timeout/
│       ├── brainstorm.md                 # Committed (shareable)
│       ├── brainstorm-review.local.md    # Local
│       ├── plan.md                       # Committed (shareable)
│       ├── plan-review.local.md          # Local
│       ├── execution-log.local.md        # Local
│       └── errors.local.md               # Local (if any)
├── state.local.json                      # Local (current workflow)
├── errors.local.md                       # Local (central error log)
├── config.json                           # Committed (team defaults)
└── config.local.json                     # Local (personal overrides)
```

**Committed files:** brainstorm.md, plan.md, config.json
**Local files (.local suffix):** reviews, logs, state, errors

**Gitignore pattern:**
```
.claude/vrau/**/*.local.md
.claude/vrau/**/*.local.json
```

### state.local.json
```json
{
  "branch": "fix/payment-api-timeout",
  "workflow": "2026-01-11-fix-payment-api-timeout",
  "state": "executing",
  "workspaceType": "new_branch",
  "startedAt": "2026-01-11T10:30:00Z"
}
```

### config.json (defaults)
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

---

## Commands

**9 Commands:**

| Command | Description | Available States |
|---------|-------------|------------------|
| `/vrau` | Auto-detect state, continue workflow | Any |
| `/vrau:start` | Begin new workflow | `idle` only |
| `/vrau:status` | Show current state and details | Any |
| `/vrau:brainstorm` | Continue/refine brainstorm | `brainstorming`, `brainstorm_review` |
| `/vrau:plan` | Continue/refine plan | `planning`, `plan_review` |
| `/vrau:execute` | Continue execution | `executing` |
| `/vrau:config` | View/edit configuration | Any |
| `/vrau:reset` | Clear state (keeps files) | Any except `idle` |
| `/vrau:abort` | Abandon workflow completely | Any except `idle` |

### `/vrau` Auto-routing Logic

| State | Action |
|-------|--------|
| `idle` | Suggest `/vrau:start` |
| `brainstorming` | Continue with `superpowers:brainstorming` |
| `brainstorm_review` | Present review options |
| `planning` | Continue with `superpowers:writing-plans` |
| `plan_review` | Present review options |
| `executing` | Continue execution / show progress |
| `done` | Present cleanup options |

### `/vrau:status` Output
```
┌─────────────────────────────────────────────┐
│ VRAU Status                                 │
├─────────────────────────────────────────────┤
│ State:    executing                         │
│ Branch:   fix/payment-api-timeout           │
│ Workflow: 2026-01-11-fix-payment-api-timeout│
│ Started:  2026-01-11 10:30:00               │
│ Workspace: new branch (no worktree)         │
│                                             │
│ Progress: Task 2/5 in progress              │
│                                             │
│ Files:                                      │
│   ✓ brainstorm.md                           │
│   ✓ plan.md                                 │
│   ◐ execution-log.local.md                  │
└─────────────────────────────────────────────┘
```

### `/vrau:config` Flow
```
/vrau:config

Current configuration:

  Storage:
    workflows: .claude/vrau/workflows/

  Reviewer:
    lenses: security, architecture, testability, maintainability

  Error Handling:
    onBrainstormError: stop_and_ask
    onPlanError: stop_and_ask
    onExecutionError: auto_retry_once
    maxRetries: 1

Options:
1. Edit setting
2. Reset to defaults
3. Show config file path
4. Done

Choose:
```

---

## Plugin Structure

**Repository Layout:**
```
vrau/
├── .claude-plugin/
│   └── plugin.json              # Plugin manifest
├── commands/
│   ├── vrau.md                  # Main auto-routing command
│   └── vrau/
│       ├── start.md
│       ├── status.md
│       ├── brainstorm.md
│       ├── plan.md
│       ├── execute.md
│       ├── config.md
│       ├── reset.md
│       └── abort.md
├── skills/
│   └── vrau/
│       └── SKILL.md             # Model-invoked skill
├── agents/
│   └── vrau-reviewer.md         # Configurable reviewer agent
├── README.md
├── LICENSE
└── CHANGELOG.md
```

### plugin.json
```json
{
  "name": "vrau",
  "version": "1.0.0",
  "description": "Workflow orchestration with autonomous execution and user-driven decisions",
  "author": {
    "name": "mguilarducci"
  },
  "repository": "https://github.com/mguilarducci/vrau",
  "license": "MIT",
  "keywords": ["workflow", "orchestration", "brainstorm", "planning"],
  "dependencies": {
    "plugins": ["superpowers"]
  },
  "commands": "./commands/",
  "skills": "./skills/",
  "agents": "./agents/"
}
```

### SKILL.md (model-invoked)

The skill watches for complex tasks and suggests `/vrau:start` - user decides.

```yaml
---
name: vrau-workflow
description: |
  Suggests vrau workflow for complex multi-step tasks.
  Use when detecting features, refactors, or tasks that
  would benefit from brainstorming → planning → execution.
user-invocable: false
---

# Vrau Workflow Suggestion

When you detect a complex multi-step task that would benefit from structured
brainstorming, planning, and execution, suggest using vrau.

## When to Suggest

- New features with multiple components
- Refactoring efforts spanning multiple files
- Bug fixes requiring investigation and multi-step resolution
- Tasks the user describes with multiple requirements

## How to Suggest

Ask the user:

"This looks like a multi-step task. Would you like to use vrau
to orchestrate this with brainstorming → planning → execution?

1. Yes, start vrau workflow
2. No, just help me directly"

If yes, invoke `/vrau:start`.
```

---

## Error Handling

**Configurable Behaviors:**

| Behavior | Description |
|----------|-------------|
| `stop_and_ask` | Pause, show error, present options (retry/skip/abort) |
| `auto_retry_once` | Retry automatically once, then stop and ask |
| `log_and_continue` | Log error, continue to next step if possible |

**Error Options When Stopped:**
```
⚠ Error in Task 3: Add retry logic

Error: Test failed - connection mock missing

Options:
1. Retry task
2. Skip task and continue
3. View full error details
4. Abort workflow

Choose:
```

**Error Logging (dual):**
- Workflow-specific: `workflows/2026-01-11-.../errors.local.md`
- Central: `.claude/vrau/errors.local.md`

---

## Completion Flow

```
✓ Workflow Complete!

Branch: fix/payment-api-timeout
Duration: 45 minutes
Tasks completed: 5/5

Invoking superpowers:finishing-a-development-branch...

[Superpowers presents merge/PR/cleanup options]

───────────────────────────────────

Workflow files cleanup:

1. Keep files
   → Workflow folder stays for reference

2. Archive
   → Move to .claude/vrau/archive/

3. Delete
   → Remove workflow folder entirely

Choose:
```

**After cleanup:**
- `state.local.json` reset to `idle`
- Ready for next `/vrau:start`

---

## Implementation Notes

### State Management
- State tracked in explicit `state.local.json` (not inferred from files)
- State file is small (~100 bytes), reset on workflow completion
- No accumulation over time

### Platform Compatibility
- No bash scripts required (unlike original plan)
- Pure markdown commands and skills
- Works on all platforms Claude Code supports

### User Configuration
- `config.json` for team defaults (committed)
- `config.local.json` for personal overrides (not committed)
- `/vrau:config` command for interactive editing

### Review System
- User always chooses review type (not automatic)
- Subagent reviewer runs in fresh context (no bias)
- Lenses are configurable per project

---

## Summary Table

| Component | Decision |
|-----------|----------|
| **Type** | Distributable plugin via marketplace |
| **Dependency** | Requires Superpowers |
| **Philosophy** | Autonomous execution, user-driven decisions |
| **States** | 7: idle → brainstorming → brainstorm_review → planning → plan_review → executing → done |
| **Commands** | 9 commands for full control |
| **Skill** | Model-invoked, suggests vrau for complex tasks |
| **Start options** | Worktree / Current branch / New branch |
| **Review options** | Checklist / Subagent / Both / Skip |
| **Reviewer** | Configurable lenses (default 4) |
| **Execution** | Subagent or parallel, autonomous |
| **Storage** | `.claude/vrau/` with timestamp-based workflow folders |
| **Committed** | brainstorm.md, plan.md, config.json |
| **Local** | reviews, logs, state, errors (`.local` suffix) |
| **Errors** | Configurable behavior, dual logging |
| **Cleanup** | User chooses: keep / archive / delete |

---

## Next Steps

Use `superpowers:writing-plans` to create a detailed implementation plan from this design.

The plan should create all files in the plugin structure:
1. `.claude-plugin/plugin.json`
2. All 9 command files in `commands/`
3. The skill file in `skills/vrau/SKILL.md`
4. The reviewer agent in `agents/vrau-reviewer.md`
5. README.md and LICENSE
