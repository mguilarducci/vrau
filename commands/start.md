---
description: "Begin a new vrau workflow"
---

# Start Vrau Workflow

## Step 0: Branch Setup (haiku)

### 0.1 Update Default Branch

```bash
git fetch origin
git checkout main
git pull origin main
```

### 0.2 Branch/Worktree Selection

```
How do you want to work?

1. Create worktree (isolated workspace)
   → Uses superpowers:using-git-worktrees
   → Best for: larger features, parallel work

2. Work in current branch
   → Stay in current directory
   → Best for: continuing existing work

3. Create new branch (no worktree)
   → git checkout -b <branch-name>
   → Best for: standard git flow
```

Based on choice:
- **Option 1**: Invoke `superpowers:using-git-worktrees` with branch name
- **Option 2**: No git operations needed
- **Option 3**: Run `git checkout -b <branch-name>`

### 0.3 Push New Branch

```bash
git push -u origin <branch-name>
```

---

## Step 1: Configuration (haiku)

### 1.1 Get Task Description

```
What are you working on?
> [Wait for user to describe task]
```

### 1.2 Generate Branch Name

Generate suggested branch name from task:
- Use kebab-case
- Prefix with fix/, feat/, refactor/, etc.
- Replace slashes with dashes for folder naming

```
Suggested branch: <generated-name>

1. Confirm
2. Edit name
```

Branch names must match: `^[a-zA-Z0-9/_-]+$`

### 1.3 Document Approach Selection

```
How should I store design documents?

A. File-based (default)
   → docs/designs/{date}-{branch}/design/{file}.md
   → Committed and pushed

B. GitHub Issues
   → Create issues for design documentation
   → Save links to docs/designs/{date}-{branch}/README.md

C. Local-only (NO COMMITS)
   → Same as A, but NEVER commit design files
   → Creates .no-commit.local marker
```

**Folder naming:** Replace `/` with `-` in branch name

**After selection:**
- Record choice in README.md
- If option C: Create `.no-commit.local` file (gitignored)

### 1.4 Model Selection

Show default models:

```
Model configuration (defaults shown):

Phase 1 - Brainstorm:
  - Branch setup: haiku
  - Instructions: haiku
  - Verifications: haiku
  - Brainstorm: opus
  - Review: opus
  - Breakdown: sonnet

Phase 2 - Plan:
  - Pre-plan: haiku
  - Write plan: opus
  - Plan review: opus

Phase 3 - Execute:
  - Simple code: haiku
  - Complex code: sonnet
  - Very complex: opus (will ask first)
  - Quality/review: sonnet (minimum)

Accept defaults? [Y/n] or specify changes
```

**Model override policy:** Locked at start. User can request change mid-flow, but only if they explicitly ask.

### 1.5 Create Workflow Folder

```bash
WORKFLOW=$(date +%Y-%m-%d)-<branch-name-with-dashes>
mkdir -p docs/designs/$WORKFLOW/design
mkdir -p docs/designs/$WORKFLOW/plan
```

### 1.6 Create README from Template

Copy `.claude/vrau/templates/README.template.md` to `docs/designs/$WORKFLOW/README.md`

Fill in template variables:
- {{TASK_TITLE}}: From task description
- {{TASK_DESCRIPTION}}: Full description
- {{BRANCH_NAME}}: Actual branch name
- {{DATE}}: Today's date
- {{DOC_APPROACH}}: A, B, or C
- {{FOLDER_NAME}}: Workflow folder name
- {{NO_COMMIT_MODE}}: true/false
- All model selections

### 1.7 Initial Commit

If NOT in no-commit mode:
```bash
git add docs/designs/$WORKFLOW/README.md
git commit -m "chore: initialize vrau workflow for <task>"
git push
```

---

## Phase 1: Brainstorm

Read [_phases/brainstorm.md](_phases/brainstorm.md) for the complete brainstorm process.

---

## Phase 2: Plan

Read [_phases/plan.md](_phases/plan.md) for the complete planning process.

---

## Phase 3: Execute

Read [_phases/execute.md](_phases/execute.md) for execution options.

---

## State Recovery

| Folder Contents | State | Resume with |
|-----------------|-------|-------------|
| README.md only | Not started | `/vrau:brainstorm` |
| README.md + design/*.md | Brainstorm done | `/vrau:plan` |
| + plan/*.md | Plan done | `/vrau:execute` |
| Execution complete in README | Done | `/vrau:status` |
