---
name: review-step-spawn-reviewer
description: Use when spawning reviewer agent - enforces sonnet model and MANDATORY separate reviewer spawn for unbiased feedback
---

# Review Step: Spawn Reviewer Agent

**This skill enforces sonnet model usage by always dispatching to sonnet.**

## Model Enforcement

**ALWAYS dispatch a Task with sonnet model.** Do not attempt to check your current model.

**Before dispatching:** Read `docs/designs/<workflow>/execution-log.md` and include its content in the prompt so the subagent has full workflow context.

```
Task tool:
- subagent_type: "general-purpose"
- model: "sonnet"
- description: "Spawn reviewer and handle feedback"
- prompt: "You are in the review phase of a vrau workflow.

DOCUMENT TO REVIEW: [specify brainstorm.md or plan.md]
DOCUMENT PATH: [full path]
EXECUTION LOG: [paste execution log content]

---

## CRITICAL: YOU MUST SPAWN A SEPARATE REVIEWER

**You are NOT the reviewer. You are the orchestrator.**

Your ONLY job is to:
1. Read the document
2. Spawn a SEPARATE vrau-reviewer agent
3. Wait for its feedback
4. Process the feedback

**WHY THIS MATTERS:**
- You have context from the workflow that biases your judgment
- A fresh reviewer agent has NO prior context
- Fresh eyes catch issues you would rationalize away
- This is the entire point of the review step

**RED FLAGS - If you think any of these, STOP:**
- 'I can review this myself' → NO. Spawn the reviewer.
- 'Spawning seems redundant' → NO. That's the bias talking.
- 'I already know this is good' → NO. That's exactly why you can't review.
- 'Let me just check a few things' → NO. You are not the reviewer.

---

## INSTRUCTIONS

### Step 1: Read the Document
Use Read tool on the document path.

### Step 2: SPAWN REVIEWER (MANDATORY)

**This is not optional. You MUST use the Task tool to spawn vrau-reviewer.**

Task tool:
- subagent_type: 'vrau:vrau-reviewer'
- model: 'sonnet'
- prompt:
  1. Task: <one-line description>
  2. Complexity: <trivial/simple/moderate/complex>
  3. Content: <paste FULL document content>
  4. Request: 'Review this <brainstorm/plan> for <completeness, clarity, technical soundness / completeness and feasibility>'

**DO NOT skip this step. DO NOT review the document yourself.**

### Step 3: Wait for Reviewer Feedback

The reviewer will return one of:
- APPROVED - Continue to next phase
- REVISE - Minor issues to fix
- RETHINK - Major issues, needs rework

### Step 4: Process Feedback

**If REVISE or RETHINK:**
- Invoke appropriate receiving-review skill:
  - For brainstorms: Skill tool with 'vrau:receiving-brainstorm-review'
  - For plans: Skill tool with 'vrau:receiving-plan-review'
- Process each issue with technical verification
- Update document
- Save, commit, push
- **Re-spawn reviewer** (go to Step 2) - NEVER skip re-review

**If APPROVED:**
- Review complete, report completion

### Step 5: Loop Constraints

Maximum 3 iterations, then ask user how to proceed."
```

**Wait for the task to complete.** Returns when review is APPROVED or user intervention needed.
