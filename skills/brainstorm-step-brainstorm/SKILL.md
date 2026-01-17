---
name: brainstorm-step-brainstorm
description: Use when executing brainstorming step in brainstorm phase - enforces opus model usage
---

# Brainstorm Step: Invoke Brainstorming Skill

**This skill enforces opus model usage. You cannot skip this check.**

## Model Enforcement Check

**FIRST: Check your current session model.**

Are you running on **opus** model right now?

### If YES (you are opus):

Execute the brainstorming step directly in this session. Skip to "Brainstorming Instructions" below.

### If NO (you are haiku or sonnet):

**You MUST dispatch a Task tool with opus model.** Do NOT execute in current session.

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

**STOP HERE.** Wait for the task to complete, then continue to next step (Save Brainstorm Output).

---

## Brainstorming Instructions

**(Only execute if you are opus model)**

### Invoke Superpowers Brainstorming Skill

```
Skill tool:
- skill: "superpowers:brainstorming"
- args: <full task description>
```

**Let the skill drive the conversation:**
- It will ask clarifying questions
- Answer thoroughly and thoughtfully
- The skill will produce a final brainstorm document when complete

**Completion signal:** The skill produces a summary or "brainstorm complete" message.

**Output:** The brainstorm document content (will be saved in next step).
