---
name: execute-phase
description: Use when executing Phase 3 of a vrau workflow - after plan is complete and approved, ready to implement
---

# Phase 3: Execute

**FIRST: Recommend session compaction**

Tell user: "Consider compacting or starting a fresh session for execution."

---

## Stateless Execution Pattern

**Each step is stateless.** Subagents have no memory of previous steps. Use the execution log to persist context.

**Execution log location:** `docs/designs/<workflow>/execution-log.md`

**Every step MUST:**
1. READ execution log at start (pass content to subagent)
2. DO the step's work
3. WRITE updated execution log with current status and context for next step
4. COMMIT AND PUSH immediately

```bash
git add docs/designs/<workflow>/execution-log.md
git commit -m "vrau(<workflow>): update execution log - <step name>"
git push
```

**When dispatching subagents:** Include execution log content in the prompt so the subagent has full context.

### Model Selection for Phase 3

| Step | Model | Rationale |
|------|-------|-----------|
| Pre-execution checks | sonnet | Dependency analysis |
| Execution | **per task** | From plan's Model field for each task |
| Post-execution | haiku | Simple git/PR operations |

---

## Pre-Execution Checks (sonnet)

**Model enforcement:** Always dispatch Task tool with sonnet model:
```
Task(subagent_type="general-purpose", model="sonnet", prompt="[Pre-Execution Checks instructions]")
```

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

**After pre-execution checks, update execution log:**
- Set Execution Status to `in_progress`
- Update Last Updated
- Commit and push

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

**After each parallel group completes, update execution log:**
- Update Current Group to next group letter
- Add completed task numbers to Completed Tasks
- Note any issues in Notes
- Update Last Updated
- Commit and push

---

## Verification (MANDATORY)

**CRITICAL: NEVER claim completion without verification.**

**BEFORE opening PR or claiming completion:**

1. **Announce to user:**
   > "I'm using the **superpowers:verification-before-completion** skill to verify all tests pass and the implementation is complete."

2. **Invoke the skill:**
   ```
   Skill tool:
   - skill: "superpowers:verification-before-completion"
   ```

3. **Wait for verification results** - the skill will:
   - Run the test suite
   - Build the project
   - Check for errors
   - Provide actual evidence of success or failure

4. **If verification FAILS:**
   - Fix issues
   - Re-verify using the skill again
   - DO NOT proceed to Post-Execution until verification passes

**Red Flags - STOP if you think any of these:**
- "Tests passed earlier, no need to verify again"
- "I already ran tests manually"
- "Verification is just a formality"
- "I'll verify after creating the PR"

**All of these mean: Stop. Use verification-before-completion skill NOW.**

---

## Post-Execution (haiku)

**Model enforcement:** Always dispatch Task tool with haiku model:
```
Task(subagent_type="general-purpose", model="haiku", prompt="[Post-Execution instructions]")
```

### Update README

Add execution summary to README.md:
```markdown
## Execution Log

**Completed:** YYYY-MM-DD
**Tasks completed:** N/N
**Notes:** [Any relevant notes]
```

### Delete Execution Log (BEFORE PR)

**IMPORTANT:** Delete the execution log file BEFORE creating the PR, so the deletion is included in the PR changes:

```bash
rm docs/designs/<workflow>/execution-log.md
git add docs/designs/<workflow>/execution-log.md
```

**Why delete before PR?** The execution log is a temporary file for workflow recovery. Including its deletion in the final PR keeps the commit history clean (single merge commit) rather than having a separate cleanup commit after merge.

### Check for GitHub Issue (Doc Approach B)

1. **Read the workflow README** to check Doc Approach:
   ```bash
   cat docs/designs/<workflow>/README.md
   ```

2. **If Doc Approach is B**, extract the issue number from the `### GitHub Issues` section:
   - Look for pattern `- #<number>:` or `- #<number>`
   - Store the issue number for the PR description

### Open PR

1. **Commit all changes** (including execution log deletion):
   ```bash
   git add -A
   git commit -m "vrau(<workflow>): complete execution"
   ```

2. **Create PR with appropriate body:**

   **If Doc Approach B (has GitHub issue):**
   ```bash
   gh pr create --title "<task description>" --body "$(cat <<'EOF'
   ## Summary
   - Implements design from docs/designs/<workflow>/
   - Plan: docs/designs/<workflow>/plan/<plan>.md

   ## Verification
   - Verified using superpowers:verification-before-completion
   - All tests passing
   - Build successful

   Closes #<issue-number>
   EOF
   )"
   ```

   **If Doc Approach A or C (no GitHub issue):**
   ```bash
   gh pr create --title "<task description>" --body "$(cat <<'EOF'
   ## Summary
   - Implements design from docs/designs/<workflow>/
   - Plan: docs/designs/<workflow>/plan/<plan>.md

   ## Verification
   - Verified using superpowers:verification-before-completion
   - All tests passing
   - Build successful
   EOF
   )"
   ```

3. **Add review request comment:**
   ```bash
   gh pr comment <pr-number> --body "@claude, review"
   ```

## Phase 3 Complete

Workflow is complete. Report to user:

> "Vrau workflow complete! PR created and ready for review."

**Note:** The execution log was deleted as part of the PR changes. No separate cleanup commit is needed after merge.
