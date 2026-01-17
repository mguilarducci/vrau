# Execution Log Pattern

**Purpose:** Enable stateless step execution by persisting workflow context to a file.

## Location

```
docs/designs/<workflow>/execution-log.md
```

Each workflow has its own execution log. Deleted at workflow completion.

## Format

```markdown
# Execution Log: <workflow>

## Workflow Context
- **Task:** <original task description>
- **Phase:** brainstorm | plan | execute
- **Branch:** <current branch name>
- **Started:** <timestamp>

## Research Summary
<brief summary of MCP tools research, if completed>

## Brainstorm
- **Status:** pending | in_progress | complete | approved
- **File:** <path to brainstorm.md>
- **Summary:** <one-line summary>

## Plan
- **Status:** pending | in_progress | complete | approved
- **File:** <path to plan.md>
- **Summary:** <one-line summary>

## Execution
- **Status:** pending | in_progress | complete
- **Current Group:** <group letter>
- **Completed Tasks:** <list of task numbers>
- **Notes:** <any relevant context>

## Last Updated
<timestamp> - <what was updated>
```

## Step Contract

Every step MUST:

1. **READ** execution log at start (if exists)
2. **DO** the step's work
3. **WRITE** updated execution log with:
   - Current status
   - Any output paths
   - Context for next step
4. **COMMIT AND PUSH** immediately:
   ```bash
   git add docs/designs/<workflow>/execution-log.md
   git commit -m "vrau(<workflow>): update execution log - <step name>"
   git push
   ```

## Cleanup

**IMPORTANT:** The execution log is deleted BEFORE creating the final PR (not after merge). This ensures the deletion is included in the PR changes for a cleaner commit history.

At workflow completion (in execute-phase Post-Execution, BEFORE `gh pr create`):
```bash
rm docs/designs/<workflow>/execution-log.md
git add docs/designs/<workflow>/execution-log.md
# Then commit with other changes and create PR
```

**Why before PR?** Including the deletion in the final PR keeps the commit history clean (single merge commit) rather than having a separate cleanup commit after merge.

## Recovery

When resuming a workflow:
1. Read execution log
2. Determine current phase and step from status fields
3. Continue from that point
