---
name: brainstorm-step-research
description: Use when executing research step in brainstorm phase - enforces haiku model usage
---

# Brainstorm Step: Research Available Tools

**This skill enforces haiku model usage by always dispatching to haiku.**

## Model Enforcement

**ALWAYS dispatch a Task with haiku model.** Do not attempt to check your current model.

**Before dispatching:** Read `docs/designs/<workflow>/execution-log.md` and include its content in the prompt so the subagent has full workflow context.

```
Task tool:
- subagent_type: "general-purpose"
- model: "haiku"
- description: "Research available tools"
- prompt: "You are in the brainstorm phase of a vrau workflow.

TASK: Research available tools before brainstorming.

INSTRUCTIONS:

1. LIST AVAILABLE MCP TOOLS
   Check for:
   - Documentation fetchers (context7, claude-docs, etc.)
   - Web search capabilities
   - API explorers or code search tools

2. IDENTIFY RELEVANT TOOLS
   Ask yourself:
   - Does the task mention specific technologies? (React, Claude API, etc.)
   - Are there docs that should be fetched?
   - Would web search help find current best practices?

3. USE RELEVANT TOOLS
   - Fetch current documentation for mentioned technologies
   - Search for recent changes or deprecations
   - Gather context before brainstorming begins

4. OUTPUT SUMMARY
   Provide a brief summary of:
   - What was researched
   - Key findings
   - Any relevant documentation or patterns found

This context will be passed to the brainstorming step."
```

**Wait for the task to complete.** The output is the research summary to pass to the next step.
