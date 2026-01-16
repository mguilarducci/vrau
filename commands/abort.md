---
description: "Delete a vrau workflow"
---

# Abort Workflow

## Find Workflow

Use the `find-workflow` skill to discover and select the workflow to delete.

If no workflows exist, display: "No workflows to abort."

## Confirmation

Show workflow contents and confirm:

```
Delete workflow: <workflow-name>

Contents:
  - <list files>

1. Delete
   → Remove folder and all files

2. Archive
   → Move to .claude/vrau/archive/

3. Cancel
```

## Execute

**Delete:**
```bash
rm -rf .claude/vrau/workflows/<workflow>
```

**Archive:**
```bash
mkdir -p .claude/vrau/archive
mv .claude/vrau/workflows/<workflow> .claude/vrau/archive/
```

Display: "Workflow deleted/archived."
