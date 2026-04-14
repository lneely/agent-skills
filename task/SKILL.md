---
name: task
intent: plan, task, todo, steps, checklist, decompose, track
description: Guidance on using task_plan and task_complete to decompose goals and track progress.
---

# Task Planning

Use `task_plan` to decompose a multi-step goal into an ordered checklist. Use `task_complete` to mark steps done as you finish them.

## When to plan

- The user asks you to do something with three or more distinct steps
- The work spans multiple files, tools, or phases
- You want to show the user a clear picture of what you're about to do before starting

## Workflow

1. Call `task_plan` to create the checklist
2. Rename `__todo` → `__wip` when work begins (via `file_edit` or shell)
3. Call `task_complete` after each step finishes
4. Use `file_*` tools to add, remove, or revise steps as the plan evolves
5. Rename `__wip` → `__done` when the goal is fully realized

## How to use

```
execute_tool: task_plan  (or use the built-in task server directly)
args: ["<goal>", "<step1>", "<step2>", ...]
```

```
execute_tool: task_complete
args: ["<plan-filename>", "<step-title-substring>"]
```

`task_complete` matches the step title case-insensitively and errors if no unchecked step matches — silent skips are caught.

## Notes

- Plan files live under `$OLLIE/pl/`
- List plans: `execute_code` with `ls $OLLIE/pl/`
- Inspect a plan: `execute_code` with `cat $OLLIE/pl/<file>`
- Rename/move plans: `execute_code` with `mv $OLLIE/pl/<src> $OLLIE/pl/<dst>`
- Bulk edits: `file_edit` or `execute_code`
- `$OLLIE` is always set in the sandbox environment; use it instead of relative paths
