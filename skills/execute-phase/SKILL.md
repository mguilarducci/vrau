---
name: execute-phase
description: Use when executing Phase 3 of a vrau workflow - after plan is complete and approved, ready to implement
---

# Phase 3: Execute

**FIRST: Recommend session compaction**

Tell user: "Consider compacting or starting a fresh session for execution."

## Pre-Execution Checks (sonnet)

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

## Execution (model varies)

### Model Selection Rules

Based on task complexity (from README.md config):
- **Simple tasks**: haiku
- **Complex tasks**: sonnet
- **Very complex tasks**: **ASK** before using opus
- **Quality/review**: sonnet minimum (never haiku)

### Execution Process

1. Invoke `superpowers:subagent-driven-development`
2. After permission granted:
   - Execute without asking for each step
   - Ask only if doubts or decisions needed
3. Follow superpower skill until complete

## Post-Execution

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
