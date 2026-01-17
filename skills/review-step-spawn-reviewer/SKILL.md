---
name: review-step-spawn-reviewer
description: Use when spawning reviewer agent - enforces opus model usage for review process
---

# Review Step: Spawn Reviewer Agent

**This skill enforces opus model usage. You cannot skip this check.**

## Model Enforcement Check

**FIRST: Check your current session model.**

Are you running on **opus** model right now?

### If YES (you are opus):

Execute the review step directly in this session. Skip to "Review Instructions" below.

### If NO (you are haiku or sonnet):

**You MUST dispatch a Task tool with opus model.** Do NOT execute in current session.

```
Task tool:
- subagent_type: "general-purpose"
- model: "opus"
- description: "Spawn reviewer and handle feedback"
- prompt: "You are in the review phase of a vrau workflow.

DOCUMENT TO REVIEW: [specify brainstorm.md or plan.md]
DOCUMENT PATH: [full path]

INSTRUCTIONS:
1. Read the document
2. Spawn vrau-reviewer agent with the content
3. Wait for feedback
4. If REVISE/RETHINK: Use receiving-review skill to process feedback
5. If APPROVED: Report completion

Follow the review process from the phase skill."
```

**STOP HERE.** Wait for the task to complete.

---

## Review Instructions

**(Only execute if you are opus model)**

### 1. Read the Document

```
Read tool: <document-path>
```

### 2. Spawn Reviewer Agent

```
Task tool:
- subagent_type: "vrau:vrau-reviewer"
- model: "opus"
- prompt:
  1. Task: <one-line description>
  2. Complexity: <trivial/simple/moderate/complex>
  3. Content: <paste document content>
  4. Request: "Review this <brainstorm/plan> for <completeness, clarity, technical soundness / completeness and feasibility>"
```

### 3. Wait for Reviewer Feedback

The reviewer will return one of:
- **APPROVED** - Continue to next phase
- **REVISE** - Minor issues to fix
- **RETHINK** - Major issues, needs rework

### 4. Process Feedback

**If REVISE or RETHINK:**
- **REQUIRED SKILL:** Use appropriate receiving-review skill:
  - For brainstorms: `vrau:receiving-brainstorm-review`
  - For plans: `vrau:receiving-plan-review`
- Process each issue with technical verification
- Update document
- Save, commit, push
- Re-spawn reviewer (go to step 2)

**If APPROVED:**
- Review complete, proceed to next step in phase

### 5. Loop Constraints

Maximum 3 iterations, then ask user how to proceed.
