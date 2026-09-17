---
name: context-task
description: Use when work needs a tracked Local Task during research, planning, execution, review, or user-directed completion.
---

# Context Task

Use this skill to keep one Local Task accurate. The SQLite-backed Local Task daemon owns task metadata and status. Pi uses the managed daemon through its configured endpoint.

New tasks receive daemon-generated UUIDs. Imported IDs stay opaque. Do not create, parse, or replace an ID.

Project scope uses `YISHAN_PROJECT_ID`. If it is set, use only that project. Without it, use global tasks only.

## Task Metadata

Use `task_read` without a document to read the synthetic, read-only brief. The brief contains daemon metadata. Use `task_update` to change the title, description, new, progressing, or cancelled status, priority, or tags. It cannot set done.

Use `task_list` to filter by status, priority, workspace, or tags. Use `task_search` to search with those filters. `task_finish` is the only tool that can set done.

A tracked Local Task may contain multiple independently deliverable plan subtasks. A plan subtask is an execution unit within the tracked Local Task; its independent deliverability does not by itself require a new Local Task.

When a newly discovered issue is related to the active plan subtask, a non-scope-changing clarification may stay in scope. If the discovery changes that plan subtask's frozen scope, requires a new decision or dependency, or changes assumptions for a later plan subtask, record it in `notes` and stop the current attempt. The controller decides whether to continue and, if so, must replace the affected scope-changing plan content with a newly frozen brief before resuming. Ask the user when the change affects unapproved behavior, commitments, or tradeoffs. For an unrelated issue, ask the user whether to create a separate Local Task.

## Context Documents

The daemon provides the paths for three context documents:

- `plan` contains the current execution plan.
- `notes` contains task-specific research and decisions.
- `outcome` contains a factual completion summary.

Before execution, every plan subtask in a tracked Local Task requires a controller-created, frozen brief in `plan.md`. Each brief must state its scope, non-goals, acceptance criteria or check, and a finite execution budget. The controller is the sole authority to create or replace scope-changing plan content. Do not execute a plan subtask without its brief.

Use `task_write` to replace `notes` or `outcome`. Executors may use it in `plan` only to record execution details that remain within the applicable frozen brief; only the controller may replace scope-changing plan content. Use `task_append_note` to add research or progress to `notes`. Keep the plan coherent. Keep notes specific to this Local Task.

Start a new `notes` document with this compact, editable Markdown structure:

```markdown
# Notes

## Summary

## Decisions

## Evidence and Progress

- YYYY-MM-DD — <fact, result, or change>

## Open Questions
```

Append dated, factual entries under **Evidence and Progress**. Move resolved material into **Summary** or **Decisions** when it becomes important. Do not preserve an old format for its own sake: when the user asks to change the notes, replace or reorganize the Markdown with `task_write`. Users can also edit `notes.md` directly in the file editor.

## Completion

Use `task_finish` only when the user explicitly asks to complete the task. After explicit direction that all work is complete, perform the self-checks, prepare a factual outcome, and finish without asking again. The tool writes the outcome and marks the task done.

Do not complete a task because implementation appears complete.

## Boundaries

Do not bypass the daemon. Do not create task identifiers. Do not write the synthetic brief. Do not use `task_update` for completion.
