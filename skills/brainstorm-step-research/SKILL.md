---
name: brainstorm-step-research
description: Use when executing research step in brainstorm phase - enforces haiku model usage
---

# Brainstorm Step: Research Context

**This skill enforces haiku model usage by always dispatching to haiku.**

## Model Enforcement

**ALWAYS dispatch a Task with haiku model.** Do not attempt to check your current model.

**Before dispatching:** Read `docs/designs/<workflow>/execution-log.md` and include its content in the prompt so the subagent has full workflow context.

```
Task tool:
- subagent_type: "general-purpose"
- model: "haiku"
- description: "Research context for brainstorm"
- prompt: "You are in the brainstorm phase of a vrau workflow.

TASK: Gather context before brainstorming.

INSTRUCTIONS:

1. IDENTIFY RESEARCH NEEDS
   - What technologies does this task involve?
   - Are there docs that should be fetched?
   - Would web search help find current best practices?

2. USE AVAILABLE TOOLS
   Use your installed MCP tools and built-in capabilities to:
   - Fetch relevant documentation
   - Search for recent changes or patterns
   - Gather implementation context

3. OUTPUT SUMMARY
   Provide a brief summary of:
   - What was researched
   - Key findings
   - Relevant documentation or patterns found

This context will be passed to the brainstorming step."
```

**Wait for the task to complete.** The output is the research summary to pass to the next step.
