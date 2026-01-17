---
name: brainstorm-step-brainstorm
description: Use when executing brainstorming step in brainstorm phase - enforces opus model usage
---

# Brainstorm Step: Invoke Brainstorming Skill

**This skill enforces opus model usage by always dispatching to opus.**

## Model Enforcement

**ALWAYS dispatch a Task with opus model.** Do not attempt to check your current model.

**Before dispatching:** Read `docs/designs/<workflow>/execution-log.md` and include its content in the prompt so the subagent has full workflow context.

```
Task tool:
- subagent_type: "general-purpose"
- model: "opus"
- description: "Execute brainstorming"
- prompt: "You are in the brainstorm phase of a vrau workflow.

TASK DESCRIPTION: [paste task description from user]

RESEARCH CONTEXT: [paste research summary from Step 0]

INSTRUCTIONS:
Invoke the superpowers:brainstorming skill with this task description.

Skill tool:
- skill: 'superpowers:brainstorming'
- args: '[task description]'

The skill will:
- Ask clarifying questions
- Guide design exploration
- Produce a final brainstorm document

Let the skill drive the conversation. Answer questions thoroughly."
```

**Wait for the task to complete.** The output is the brainstorm document ready to save.
