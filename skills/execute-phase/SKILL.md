---
name: execute-phase
description: Use when executing Phase 3 of a vrau workflow - after plan is complete and approved, ready to implement
---

# Phase 3: Execute

**FIRST: Recommend session compaction**

Tell user: "Consider compacting or starting a fresh session for execution."

### Model Selection for Phase 3

| Step | Model | Rationale |
|------|-------|-----------|
| Pre-execution checks | sonnet | Dependency analysis |
| Execution | **per task** | From plan's Model field for each task |
| Post-execution | haiku | Simple git/PR operations |

---

## Pre-Execution Checks (sonnet)

**Model enforcement:** If current session is not sonnet, dispatch Task tool:
```
Task(subagent_type="general-purpose", model="sonnet", prompt="[Pre-Execution Checks instructions]")
```
If current session is sonnet, run in current session.

### Verify Dependencies

Check if all dependencies listed in plan file are implemented:

```
Checking plan dependencies...

Dependencies found:
- [ ] Task 1 depends on: (none)
- [ ] Task 3 depends on: Task 1, Task 2
- [ ] Task 5 depends on: Task 3

Missing implementations: [list any missing]
```

If dependencies missing, notify user before proceeding.

### Ask Permission

```
Plan approved. Ready to execute.

Proceed with execution? [Y/n]
```

## Execution (model per task from plan)

**Model enforcement:** The vrau-executing-plans skill handles per-task model selection from the plan. No model enforcement needed at this level - the skill will dispatch tasks with their specified models.

### Execution Process

1. Invoke `vrau:vrau-executing-plans`
2. Skill reads parallel groups from plan
3. For groups with 2+ tasks: dispatches parallel agents
4. For single-task groups: uses sequential execution
5. Uses model specified per task in plan

**Parallel execution:**
- Tasks in same parallel group run concurrently
- Wait for group completion before next group
- Model enforced per task from plan's Model field

## Post-Execution (haiku)

**Model enforcement:** If current session is not haiku, dispatch Task tool:
```
Task(subagent_type="general-purpose", model="haiku", prompt="[Post-Execution instructions]")
```
If current session is haiku, run in current session.

### Update README

Add execution summary to README.md:
```markdown
## Execution Log

**Completed:** YYYY-MM-DD
**Tasks completed:** N/N
**Notes:** [Any relevant notes]
```

### Open PR

1. Open PR with `gh pr create`
2. Add comment: `@claude, review`

```bash
gh pr create --title "<task description>" --body "## Summary
- Implements design from docs/designs/<workflow>/
- Plan: docs/designs/<workflow>/plan/<plan>.md

## Test Plan
- [x] All tests pass
- [x] Manual verification complete"

gh pr comment <pr-number> --body "@claude, review"
```

## Phase 3 Complete

Workflow is complete. Report to user:

> "Vrau workflow complete! PR created and ready for review."
