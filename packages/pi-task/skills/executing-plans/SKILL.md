---
name: executing-plans
description: Use when you have an approved plan and can execute it in the current session without the full subagent workflow.
---

# Executing Plans

Use this skill to execute an approved plan in the current session.

The SQLite-backed Local Task daemon owns task metadata and status. Pi uses the managed daemon through its configured endpoint. New tasks use daemon-generated UUIDs. Imported IDs stay opaque.

## Load the Task

Use `task_read` for Local Task metadata and its synthetic, read-only brief. That brief is not a controller-frozen plan-subtask brief, and executors cannot write it. A tracked Local Task may contain multiple independently deliverable plan subtasks in `plan.md`; their independent deliverability does not by itself require separate Local Tasks. Every plan subtask must have an actionable frozen brief before execution. Each brief must define its scope, non-goals, acceptance criteria or check, and a finite execution budget. Read `plan.md` and `notes` when they are relevant. Use `task_list` or `task_search` to find work by status, priority, workspace, or tags.

If `YISHAN_PROJECT_ID` is set, work only with that project. Without it, work only with global tasks.

## Execute the Plan

The controller owns plan-subtask scope and is the sole authority to create or replace scope-changing plan content. Local Task metadata identifies and tracks the Local Task; `plan.md` contains the frozen, independently verifiable briefs for its plan subtasks. Execute only when the applicable plan-subtask brief defines its scope, non-goals, acceptance criteria or check, and finite execution budget.

1. Read `plan.md`, including the applicable frozen plan-subtask brief, before you edit.
2. Confirm the plan subtask is independently deliverable; do not create a new Local Task solely for that reason.
3. Perform one frozen plan subtask at a time, staying within its scope boundary and execution budget.
4. Run its acceptance check and focused verification.
5. Record material discoveries in `notes` as dated, factual entries under `## Evidence and Progress`. Keep the document readable with `## Summary`, `## Decisions`, and `## Open Questions`; use `task_write` to reorganize it when needed.

Do not incorporate scope-changing related work into an active plan subtask. If a discovery requires work outside the frozen brief, stop and report the boundary change to the controller. The controller decides whether to continue and, if so, must replace the affected scope-changing plan content with a newly frozen brief before additional or revised work proceeds. Do not expand or retry a stopped plan subtask indefinitely; replan only through a new frozen brief with a new finite execution budget.

Use `task_write` to update `plan.md` only for execution details that remain within the frozen brief; do not modify the frozen brief or other scope-changing plan content. Scope-changing work requires the controller to replace the applicable plan content. Use `task_append_note` to add task-specific research or progress. Use `task_update` for a title, description, new, progressing, or cancelled status, priority, or tags. It cannot set done.

`task_update` cannot complete a task. Use `task_finish` only when the user explicitly asks to complete the task. When the user explicitly says all work is complete, run the self-checks and finish without asking again. It writes `outcome` and marks the task done.

## Boundaries

Do not bypass the daemon. Do not create or parse task IDs. Do not write the synthetic Local Task brief. Do not change a frozen brief while executing it. Stop when the applicable plan-subtask brief no longer fits the work, its acceptance check cannot be met within budget, or completing it would cross a non-goal. The controller decides whether to continue; if it does, replan only through a newly controller-frozen `plan.md` brief. Local Task closure remains under the user's explicit control.
