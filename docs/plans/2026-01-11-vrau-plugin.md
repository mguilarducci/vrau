# Vrau Plugin Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Create a distributable Claude Code plugin that orchestrates workflow (brainstorm → plan → execute) with autonomous execution and user-driven decisions.

**Architecture:** Plugin follows Claude Code plugin structure with `.claude-plugin/plugin.json` manifest, `commands/` for user-invoked commands, `skills/` for model-invoked suggestions, and `agents/` for the configurable reviewer. Vrau orchestrates the flow while delegating to Superpowers skills for heavy lifting.

**Tech Stack:** Markdown files for commands/skills/agents, JSON for plugin manifest and config files.

---

## Task 1: Create Plugin Manifest

**Files:**
- Create: `.claude-plugin/plugin.json`

**Step 1: Create plugin directory**

Run: `mkdir -p .claude-plugin`
Expected: Directory created

**Step 2: Write plugin.json**

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

**Step 3: Commit**

```bash
git add .claude-plugin/plugin.json
git commit -m "feat: add plugin manifest with superpowers dependency"
```

---

## Task 2: Create Main /vrau Command (Auto-Router)

**Files:**
- Create: `commands/vrau.md`

**Step 1: Create commands directory**

Run: `mkdir -p commands`
Expected: Directory created

**Step 2: Write vrau.md**

```markdown
---
description: "Continue vrau workflow - auto-detects current state and takes appropriate action"
---

# Vrau Workflow

Read the current state from `.claude/vrau/state.local.json`.

## State Routing

Based on the `state` field:

| State | Action |
|-------|--------|
| File not found OR `idle` | Tell user: "No active workflow. Use `/vrau:start` to begin a new workflow." |
| `brainstorming` | Continue with `superpowers:brainstorming` skill. When complete, transition to `brainstorm_review`. |
| `brainstorm_review` | Present review options (see Review Options below) |
| `planning` | Continue with `superpowers:writing-plans` skill. When complete, transition to `plan_review`. |
| `plan_review` | Present review options (see Review Options below) |
| `executing` | Show execution progress from `execution-log.local.md`. Ask if user wants to continue execution. |
| `done` | Present cleanup options (Keep / Archive / Delete) |

## Review Options

Present these options:

```
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
```

After review choice is made:
- If `brainstorm_review`: Update state to `planning`, invoke `superpowers:writing-plans`
- If `plan_review`: Update state to `executing`, present execution options

## Execution Options

After plan is approved:

```
Plan approved! Ready to execute.

How do you want to run the plan?

1. Subagent-driven (this session)
   → Uses superpowers:subagent-driven-development
   → Fresh subagent per task
   → Autonomous execution

2. Parallel session (separate terminal)
   → Uses superpowers:executing-plans
   → Open new terminal in worktree/branch
   → Return here when done
```

## State File Format

```json
{
  "branch": "fix/payment-api-timeout",
  "workflow": "2026-01-11-fix-payment-api-timeout",
  "state": "executing",
  "workspaceType": "new_branch",
  "startedAt": "2026-01-11T10:30:00Z"
}
```
```

**Step 3: Commit**

```bash
git add commands/vrau.md
git commit -m "feat: add main /vrau auto-routing command"
```

---

## Task 3: Create /vrau:start Command

**Files:**
- Create: `commands/vrau/start.md`

**Step 1: Create vrau subdirectory**

Run: `mkdir -p commands/vrau`
Expected: Directory created

**Step 2: Write start.md**

```markdown
---
description: "Begin a new vrau workflow"
---

# Start Vrau Workflow

First, check if `.claude/vrau/state.local.json` exists and has a state other than `idle`. If so:
- Tell user: "A workflow is already in progress. Use `/vrau:status` to see details or `/vrau:abort` to abandon it."
- Stop here.

## Step 1: Task Description

Ask the user:

```
What are you working on?
> [Wait for user to describe task in one sentence]
```

## Step 2: Branch Name

Generate a suggested branch name from the task description (kebab-case, prefix with fix/, feat/, refactor/, etc.).

Present:

```
Suggested branch: <generated-name>

1. Confirm
2. Edit name
```

Branch names must match: `^[a-zA-Z0-9/_-]+$`

## Step 3: Workspace Setup

Present options:

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
```

Based on choice:
- **Option 1**: Invoke `superpowers:using-git-worktrees` with branch name
- **Option 2**: No git operations
- **Option 3**: Run `git checkout -b <branch-name>`

## Step 4: Initialize Workflow

1. Create directories:
   ```bash
   mkdir -p .claude/vrau/workflows/$(date +%Y-%m-%d)-<branch-name>
   ```

2. Create `state.local.json`:
   ```json
   {
     "branch": "<branch-name>",
     "workflow": "YYYY-MM-DD-<branch-name>",
     "state": "brainstorming",
     "workspaceType": "<worktree|current_branch|new_branch>",
     "startedAt": "<ISO-timestamp>"
   }
   ```

3. Ensure `.claude/vrau/**/*.local.md` and `.claude/vrau/**/*.local.json` are in `.gitignore`

## Step 5: Confirmation and Start

Display:

```
✓ Branch created: <branch-name>
✓ Workflow initialized
✓ State: brainstorming

Starting brainstorm phase...
```

Then invoke `superpowers:brainstorming` skill with the task description.
```

**Step 3: Commit**

```bash
git add commands/vrau/start.md
git commit -m "feat: add /vrau:start command with workspace options"
```

---

## Task 4: Create /vrau:status Command

**Files:**
- Create: `commands/vrau/status.md`

**Step 1: Write status.md**

```markdown
---
description: "Show current vrau workflow status"
---

# Vrau Status

Read `.claude/vrau/state.local.json`. If not found, display:

```
No active vrau workflow.
Use /vrau:start to begin.
```

Otherwise, display status:

```
┌─────────────────────────────────────────────┐
│ VRAU Status                                 │
├─────────────────────────────────────────────┤
│ State:    <state>                           │
│ Branch:   <branch>                          │
│ Workflow: <workflow>                        │
│ Started:  <startedAt>                       │
│ Workspace: <workspaceType>                  │
│                                             │
│ Files:                                      │
│   <✓|○> brainstorm.md                       │
│   <✓|○> plan.md                             │
│   <✓|○> execution-log.local.md              │
└─────────────────────────────────────────────┘
```

Check for file existence:
- `✓` = file exists
- `○` = file does not exist

If state is `executing`, also show progress from `execution-log.local.md`:
- Count completed tasks
- Show current task if in progress
```

**Step 2: Commit**

```bash
git add commands/vrau/status.md
git commit -m "feat: add /vrau:status command"
```

---

## Task 5: Create /vrau:brainstorm Command

**Files:**
- Create: `commands/vrau/brainstorm.md`

**Step 1: Write brainstorm.md**

```markdown
---
description: "Continue or refine the current brainstorm"
---

# Vrau Brainstorm

Read `.claude/vrau/state.local.json`.

## Validation

If state is not `brainstorming` or `brainstorm_review`:
- Tell user: "Cannot brainstorm in current state: <state>. Use `/vrau:status` to see details."
- Stop here.

## Actions

### If state is `brainstorming`:

Continue with `superpowers:brainstorming` skill.

If the brainstorm is complete, save to workflow folder:
- `.claude/vrau/workflows/<workflow>/brainstorm.md`

Update state to `brainstorm_review`.

### If state is `brainstorm_review`:

Ask user:

```
Brainstorm is complete. What would you like to do?

1. Refine brainstorm
   → Return to brainstorming phase

2. View current brainstorm
   → Display the brainstorm.md contents

3. Proceed to review
   → Present review options
```

Based on choice:
- **Option 1**: Update state to `brainstorming`, continue with `superpowers:brainstorming`
- **Option 2**: Read and display `.claude/vrau/workflows/<workflow>/brainstorm.md`
- **Option 3**: Present review options (checklist / subagent / both / skip)
```

**Step 2: Commit**

```bash
git add commands/vrau/brainstorm.md
git commit -m "feat: add /vrau:brainstorm command"
```

---

## Task 6: Create /vrau:plan Command

**Files:**
- Create: `commands/vrau/plan.md`

**Step 1: Write plan.md**

```markdown
---
description: "Continue or refine the current plan"
---

# Vrau Plan

Read `.claude/vrau/state.local.json`.

## Validation

If state is not `planning` or `plan_review`:
- Tell user: "Cannot plan in current state: <state>. Use `/vrau:status` to see details."
- Stop here.

## Actions

### If state is `planning`:

Continue with `superpowers:writing-plans` skill.

Reference the brainstorm document at:
- `.claude/vrau/workflows/<workflow>/brainstorm.md`

When plan is complete, save to workflow folder:
- `.claude/vrau/workflows/<workflow>/plan.md`

Update state to `plan_review`.

### If state is `plan_review`:

Ask user:

```
Plan is complete. What would you like to do?

1. Refine plan
   → Return to planning phase

2. View current plan
   → Display the plan.md contents

3. Proceed to review
   → Present review options
```

Based on choice:
- **Option 1**: Update state to `planning`, continue with `superpowers:writing-plans`
- **Option 2**: Read and display `.claude/vrau/workflows/<workflow>/plan.md`
- **Option 3**: Present review options (checklist / subagent / both / skip)
```

**Step 2: Commit**

```bash
git add commands/vrau/plan.md
git commit -m "feat: add /vrau:plan command"
```

---

## Task 7: Create /vrau:execute Command

**Files:**
- Create: `commands/vrau/execute.md`

**Step 1: Write execute.md**

```markdown
---
description: "Continue execution of the current plan"
---

# Vrau Execute

Read `.claude/vrau/state.local.json`.

## Validation

If state is not `executing`:
- Tell user: "Cannot execute in current state: <state>. Use `/vrau:status` to see details."
- Stop here.

## Show Progress

Read `.claude/vrau/workflows/<workflow>/execution-log.local.md` if it exists.

Display current progress:

```
Execution Progress:
- Completed: X/Y tasks
- Current: <task name or "None">
- Errors: <count or "None">
```

## Continue Execution

Ask user:

```
How would you like to proceed?

1. Continue execution
   → Resume from current task

2. View execution log
   → Show detailed progress

3. View plan
   → Show the plan being executed

4. Abort execution
   → Stop and return to idle
```

Based on choice:
- **Option 1**: Continue with the execution method originally chosen (subagent or parallel)
- **Option 2**: Display `execution-log.local.md`
- **Option 3**: Display `plan.md`
- **Option 4**: Update state to `idle`, offer cleanup options

## Execution Completion

When all tasks complete:
1. Update state to `done`
2. Invoke `superpowers:finishing-a-development-branch`
3. Present cleanup options
```

**Step 2: Commit**

```bash
git add commands/vrau/execute.md
git commit -m "feat: add /vrau:execute command"
```

---

## Task 8: Create /vrau:config Command

**Files:**
- Create: `commands/vrau/config.md`

**Step 1: Write config.md**

```markdown
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
```

**Step 2: Commit**

```bash
git add commands/vrau/config.md
git commit -m "feat: add /vrau:config command"
```

---

## Task 9: Create /vrau:reset Command

**Files:**
- Create: `commands/vrau/reset.md`

**Step 1: Write reset.md**

```markdown
---
description: "Clear workflow state (keeps files)"
---

# Vrau Reset

Read `.claude/vrau/state.local.json`.

## Validation

If file not found or state is `idle`:
- Tell user: "No active workflow to reset."
- Stop here.

## Confirmation

Display current state and ask:

```
Current workflow: <workflow>
State: <state>
Branch: <branch>

This will clear the workflow state but keep all files.

1. Confirm reset
2. Cancel
```

## Reset Action

If user confirms:

1. Update `state.local.json`:
   ```json
   {
     "state": "idle"
   }
   ```

2. Display:
   ```
   ✓ Workflow state reset to idle
   ✓ Files preserved in: .claude/vrau/workflows/<workflow>/

   Use /vrau:start to begin a new workflow.
   ```
```

**Step 2: Commit**

```bash
git add commands/vrau/reset.md
git commit -m "feat: add /vrau:reset command"
```

---

## Task 10: Create /vrau:abort Command

**Files:**
- Create: `commands/vrau/abort.md`

**Step 1: Write abort.md**

```markdown
---
description: "Abandon workflow completely"
---

# Vrau Abort

Read `.claude/vrau/state.local.json`.

## Validation

If file not found or state is `idle`:
- Tell user: "No active workflow to abort."
- Stop here.

## Confirmation

Display current state and ask:

```
⚠ Abort Workflow

Current workflow: <workflow>
State: <state>
Branch: <branch>

What do you want to do with workflow files?

1. Keep files
   → Files stay in .claude/vrau/workflows/<workflow>/

2. Archive files
   → Move to .claude/vrau/archive/<workflow>/

3. Delete files
   → Remove workflow folder entirely

4. Cancel
   → Return without aborting
```

## Abort Action

Based on choice (1, 2, or 3):

1. Perform file action (keep/archive/delete)

2. Reset `state.local.json`:
   ```json
   {
     "state": "idle"
   }
   ```

3. Display:
   ```
   ✓ Workflow aborted
   ✓ Files: <kept|archived|deleted>
   ✓ State reset to idle

   Use /vrau:start to begin a new workflow.
   ```
```

**Step 2: Commit**

```bash
git add commands/vrau/abort.md
git commit -m "feat: add /vrau:abort command"
```

---

## Task 11: Create Vrau Skill (Model-Invoked)

**Files:**
- Create: `skills/vrau/SKILL.md`

**Step 1: Create skills directory**

Run: `mkdir -p skills/vrau`
Expected: Directory created

**Step 2: Write SKILL.md**

```markdown
---
name: vrau-workflow
description: |
  Suggests vrau workflow for complex multi-step tasks.
  Use when detecting features, refactors, or tasks that
  would benefit from brainstorming → planning → execution.
---

# Vrau Workflow Suggestion

When you detect a complex multi-step task that would benefit from structured
brainstorming, planning, and execution, suggest using vrau.

## When to Suggest

Suggest vrau for:
- New features with multiple components
- Refactoring efforts spanning multiple files
- Bug fixes requiring investigation and multi-step resolution
- Tasks the user describes with multiple requirements
- Any task that would benefit from brainstorm → plan → execute flow

Do NOT suggest vrau for:
- Simple single-file changes
- Quick fixes or typo corrections
- Questions or research tasks
- Tasks the user explicitly wants done immediately

## How to Suggest

Present the choice naturally:

```
This looks like a multi-step task that could benefit from structured workflow.

Would you like to use vrau to orchestrate this?
→ Brainstorm requirements first
→ Create a detailed plan
→ Execute with tracking and verification

1. Yes, start vrau workflow
2. No, just help me directly
```

If user chooses "Yes", invoke `/vrau:start` with the task description.

If user chooses "No", proceed normally without vrau.

## Key Principle

**Never force vrau.** Always let the user decide. Vrau is for when structure helps,
not for imposing process on simple tasks.
```

**Step 3: Commit**

```bash
git add skills/vrau/SKILL.md
git commit -m "feat: add vrau-workflow skill for model-invoked suggestions"
```

---

## Task 12: Create Vrau Reviewer Agent

**Files:**
- Create: `agents/vrau-reviewer.md`

**Step 1: Create agents directory**

Run: `mkdir -p agents`
Expected: Directory created

**Step 2: Write vrau-reviewer.md**

```markdown
---
name: vrau-reviewer
description: |
  Configurable reviewer agent for brainstorms and plans.
  Provides fresh-eyes analysis with structured feedback
  organized by severity: Critical, Important, Suggestions.
model: inherit
---

You are a Senior Technical Reviewer. Your role is to provide fresh-eyes analysis
of brainstorm documents and implementation plans.

## Review Process

1. **Read the document** provided to you completely
2. **Apply configured lenses** (see below)
3. **Categorize findings** by severity
4. **Provide structured feedback**

## Review Lenses

Apply these analysis perspectives (configurable):

### Default Lenses (always apply)

**Security:**
- Authentication/authorization considerations
- Input validation and sanitization
- Sensitive data handling
- Common vulnerability patterns

**Architecture:**
- Component boundaries and responsibilities
- Coupling and cohesion
- Integration points
- Scalability considerations

**Testability:**
- Test coverage strategy
- Mock/stub requirements
- Edge cases identified
- Test isolation

**Maintainability:**
- Code organization
- Naming clarity
- Documentation needs
- Future modification ease

### Optional Lenses (if configured)

**Performance:** Resource usage, bottlenecks, optimization opportunities
**UX:** User experience implications, workflow clarity
**Observability:** Logging, monitoring, debugging support
**Integrations:** External system dependencies, API contracts

## Output Format

```markdown
## Summary
[2-3 sentences: what this document proposes and overall assessment]

## Critical Issues
[Must be addressed before proceeding - blocking problems]

- **[Issue title]**: [Description and why it's critical]
  - Recommendation: [Specific action to take]

## Important Issues
[Should be addressed - significant but not blocking]

- **[Issue title]**: [Description]
  - Recommendation: [Specific action to take]

## Suggestions
[Nice to have - improvements and alternatives]

- [Suggestion with brief rationale]

## Verdict

[APPROVE | REVISE | RETHINK]

- APPROVE: Ready to proceed, minor suggestions only
- REVISE: Good foundation, but important issues need addressing
- RETHINK: Critical issues require significant changes
```

## Guidelines

- Be constructive, not just critical
- Acknowledge what's done well
- Provide specific, actionable recommendations
- Consider the project context
- Don't over-engineer - respect YAGNI
- Focus on what matters most
```

**Step 3: Commit**

```bash
git add agents/vrau-reviewer.md
git commit -m "feat: add vrau-reviewer agent with configurable lenses"
```

---

## Task 13: Update .gitignore for Vrau Files

**Files:**
- Modify: `.gitignore`

**Step 1: Read current .gitignore**

Current content should be `*.local.*`

**Step 2: Add vrau-specific patterns**

Add to `.gitignore`:

```
# Existing
*.local.*

# Vrau local files
.claude/vrau/**/*.local.md
.claude/vrau/**/*.local.json
```

**Step 3: Commit**

```bash
git add .gitignore
git commit -m "chore: add vrau local file patterns to gitignore"
```

---

## Task 14: Create Default Config File

**Files:**
- Create: `.claude/vrau/config.json`

**Step 1: Create vrau directory**

Run: `mkdir -p .claude/vrau`
Expected: Directory created

**Step 2: Write config.json**

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

**Step 3: Commit**

```bash
git add .claude/vrau/config.json
git commit -m "feat: add default vrau configuration"
```

---

## Task 15: Update README

**Files:**
- Modify: `README.md`

**Step 1: Write comprehensive README**

```markdown
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
```

**Step 2: Commit**

```bash
git add README.md
git commit -m "docs: comprehensive README with installation and usage"
```

---

## Task 16: Create LICENSE File (if needed update)

**Files:**
- Verify: `LICENSE` (already exists from initial commit)

**Step 1: Verify LICENSE exists**

Run: `cat LICENSE | head -20`
Expected: MIT license text

**Step 2: No changes needed**

LICENSE already exists with correct content.

---

## Task 17: Final Verification

**Step 1: Verify all files exist**

Run: `find . -name "*.md" -o -name "*.json" | grep -v node_modules | grep -v .git | sort`

Expected files:
```
./.claude/vrau/config.json
./commands/vrau.md
./commands/vrau/abort.md
./commands/vrau/brainstorm.md
./commands/vrau/config.md
./commands/vrau/execute.md
./commands/vrau/plan.md
./commands/vrau/reset.md
./commands/vrau/start.md
./commands/vrau/status.md
./agents/vrau-reviewer.md
./skills/vrau/SKILL.md
./.claude-plugin/plugin.json
./README.md
./docs/2026-01-11-vrau-plugin-brainstorm.md
./docs/plans/2026-01-11-vrau-plugin.md
```

**Step 2: Verify plugin structure**

Run: `ls -la .claude-plugin/ commands/ commands/vrau/ skills/vrau/ agents/`

Expected: All directories exist with correct files

**Step 3: Final commit**

Run: `git status`

If any uncommitted changes:
```bash
git add -A
git commit -m "chore: final plugin structure verification"
```

---

## Summary

This plan creates a complete vrau plugin with:

- **1 plugin manifest** (`.claude-plugin/plugin.json`)
- **9 command files** (`commands/vrau.md` + 8 subcommands)
- **1 skill file** (`skills/vrau/SKILL.md`)
- **1 agent file** (`agents/vrau-reviewer.md`)
- **1 config file** (`.claude/vrau/config.json`)
- **Updated .gitignore and README**

Each task is atomic with clear steps and commit points. The plugin follows Claude Code conventions and integrates with Superpowers for heavy lifting.
