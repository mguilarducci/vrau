---
description: "Delete a vrau workflow"
---

# Abort Workflow

Delete a workflow folder and its contents.

## Find Workflow

Scan `.claude/vrau/workflows/` for folders.

If no workflows exist:
```
No workflows to abort.
```

If multiple workflows exist, list them and ask which to delete:
```
Found workflows:
1. 2026-01-10-add-auth
2. 2026-01-12-fix-api
3. 2026-01-13-refactor-db

Which workflow to delete?
```

## Confirmation

Show workflow contents and confirm:

```
Delete workflow: 2026-01-10-add-auth

Contents:
  - brainstorm.md
  - plan.md

1. Delete
   -> Remove folder and all files

2. Archive
   -> Move to .claude/vrau/archive/

3. Cancel
```

## Execute

Based on choice:

**Delete:**
```bash
rm -rf .claude/vrau/workflows/<workflow>
```

**Archive:**
```bash
mkdir -p .claude/vrau/archive
mv .claude/vrau/workflows/<workflow> .claude/vrau/archive/
```

Display result:
```
Workflow deleted/archived.
```
