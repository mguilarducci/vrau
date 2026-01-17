---
name: brainstorm-step-brainstorm
description: Use when executing brainstorming step in brainstorm phase - enforces opus model usage
---

# Brainstorm Step: Invoke Brainstorming Skill

**This skill runs in the MAIN session because it requires user interaction.**

## CRITICAL: User Must Answer Questions

**Brainstorming requires USER input.** Do NOT dispatch a subagent that answers its own questions.

The orchestrator must:
1. Invoke `superpowers:brainstorming` skill directly in the MAIN session
2. Let the skill ask the USER questions
3. Wait for USER answers
4. Continue until brainstorm is complete

```
Skill tool:
- skill: "superpowers:brainstorming"
- args: "[task description]"
```

**Why not a subagent?** Subagents cannot interact with the user. If brainstorming runs in a subagent, it will either:
- Answer its own questions (WRONG - biased, no user input)
- Return incomplete (no clarification gathered)

**The brainstorming conversation happens in the main session with the user.**

**Output:** The brainstorm document ready to save.
