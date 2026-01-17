---
name: plan-step-write
description: Use when executing write plan step in plan phase - enforces opus model usage
---

# Plan Step: Write Plan

**This skill enforces opus model usage. You cannot skip this check.**

## Model Enforcement Check

**FIRST: Check your current session model.**

Are you running on **opus** model right now?

### If YES (you are opus):

Execute the write plan step directly in this session. Skip to "Write Plan Instructions" below.

### If NO (you are haiku or sonnet):

**You MUST dispatch a Task tool with opus model.** Do NOT execute in current session.

```
Task tool:
- subagent_type: "general-purpose"
- model: "opus"
- description: "Write implementation plan"
- prompt: "You are in the plan phase of a vrau workflow.

DESIGN DOCUMENT: [read and paste design document content]

INSTRUCTIONS:
Invoke the vrau:vrau-writing-plans skill to create the implementation plan.

Skill tool:
- skill: 'vrau:vrau-writing-plans'

The skill will guide you to create a plan with:
- Task dependency graph
- Parallel execution groups
- Model assignments per task
- Detailed implementation steps

Save the plan to: docs/designs/<workflow>/plan/<design-name>-plan.md

Commit after each major section:
git add docs/designs/<workflow>/plan/
git commit -m 'plan: <description> [#issue-number]'
git push"
```

**STOP HERE.** Wait for the task to complete, then continue to Review Loop.

---

## Write Plan Instructions

**(Only execute if you are opus model)**

### 1. Invoke vrau-writing-plans Skill

```
Skill tool:
- skill: "vrau:vrau-writing-plans"
```

### 2. Provide Context

Reference the selected design document from Pre-Plan Setup.

### 3. Write Plan

Save to: `docs/designs/<workflow>/plan/<design-name>-plan.md`

### 4. Commit Discipline

Commit after each major section:
```bash
git add docs/designs/<workflow>/plan/
git commit -m "plan: <description> [#issue-number]"
git push
```

**Plan requirements (vrau-writing-plans format):**
- Task dependency graph (visual ASCII)
- Parallel execution groups table
- Model assignments table
- Per-task: depends-on, parallel group, model fields
- Reference the design document
- Follow bite-sized task format
