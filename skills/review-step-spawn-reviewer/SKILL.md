---
name: review-step-spawn-reviewer
description: Use when spawning reviewer agent - enforces opus model usage for review process
---

# Review Step: Spawn Reviewer Agent

**This skill enforces opus model usage by always dispatching to opus.**

## Model Enforcement

**ALWAYS dispatch a Task with opus model.** Do not attempt to check your current model.

**Before dispatching:** Read `docs/designs/<workflow>/execution-log.md` and include its content in the prompt so the subagent has full workflow context.

```
Task tool:
- subagent_type: "general-purpose"
- model: "opus"
- description: "Spawn reviewer and handle feedback"
- prompt: "You are in the review phase of a vrau workflow.

DOCUMENT TO REVIEW: [specify brainstorm.md or plan.md]
DOCUMENT PATH: [full path]

INSTRUCTIONS:

1. READ THE DOCUMENT
   Use Read tool on the document path.

2. SPAWN REVIEWER AGENT
   Task tool:
   - subagent_type: 'vrau:vrau-reviewer'
   - model: 'opus'
   - prompt:
     1. Task: <one-line description>
     2. Complexity: <trivial/simple/moderate/complex>
     3. Content: <paste document content>
     4. Request: 'Review this <brainstorm/plan> for <completeness, clarity, technical soundness / completeness and feasibility>'

3. WAIT FOR REVIEWER FEEDBACK
   The reviewer will return one of:
   - APPROVED - Continue to next phase
   - REVISE - Minor issues to fix
   - RETHINK - Major issues, needs rework

4. PROCESS FEEDBACK
   If REVISE or RETHINK:
   - Use appropriate receiving-review skill:
     - For brainstorms: vrau:receiving-brainstorm-review
     - For plans: vrau:receiving-plan-review
   - Process each issue with technical verification
   - Update document
   - Save, commit, push
   - Re-spawn reviewer (go to step 2)

   If APPROVED:
   - Review complete, report completion

5. LOOP CONSTRAINTS
   Maximum 3 iterations, then ask user how to proceed."
```

**Wait for the task to complete.** Returns when review is APPROVED or user intervention needed.
