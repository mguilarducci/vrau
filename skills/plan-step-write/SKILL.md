---
name: plan-step-write
description: Use when executing write plan step in plan phase - enforces opus model usage
---

# Plan Step: Write Plan

**This skill enforces opus model usage by always dispatching to opus.**

## Model Enforcement

**ALWAYS dispatch a Task with opus model.** Do not attempt to check your current model.

**Before dispatching:** Read `docs/designs/<workflow>/execution-log.md` and include its content in the prompt so the subagent has full workflow context.

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

**Wait for the task to complete.** The output is the plan document saved and committed.
