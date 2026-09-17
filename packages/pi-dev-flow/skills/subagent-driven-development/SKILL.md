---
name: subagent-driven-development
description: Use when executing a multi-plan-subtask implementation plan by dispatching one fresh builder agent per plan subtask, reviewing each plan subtask before moving on, and keeping controller context narrow.
---

# Subagent-Driven Development

Use this skill to execute an approved implementation plan with isolated subagents.

## Relationship To Context Skills

When the work has a tracked Local Task, use `context-task` as the controller's durable Local Task record.

- use `task_read` without a document to read the synthetic, read-only brief
- use `task_read` to read the current `plan.md`, `notes.md`, and `outcome.md` context documents
- use `context-task` to keep `plan.md` as the current execution source of truth, store discoveries specific to the tracked Local Task in `notes.md`, and record the final outcome in `outcome.md`; tracked Local Task completion remains user-controlled

Use `context-memory` when prior durable decisions, architecture notes, or cross-Local-Task discoveries may affect the plan or plan-subtask sequencing.

## Why This Skill Exists

The goal is to keep each subagent's context narrow so its decisions stay clear, reusable, and cheaper than carrying one large implementation session.

This workflow separates roles:

- controller: coordinates the plan and handoffs
- `builder`: implements one plan subtask
- `code-reviewer`: reviews each completed plan subtask and the final broader change

## When to Use This Skill

Use this skill when:

- you already have an approved implementation plan
- the plan has multiple plan subtasks or checkpoints
- plan subtasks are independent enough to execute sequentially with clean handoffs
- you want stronger context isolation between implementation and review

Do not use this skill when the work is tiny, highly exploratory, or too tightly coupled to split into plan-subtask handoffs.

## Plan-Subtask Sizing Gate

Apply this gate before dispatching any `builder`. The controller, not the builder, owns plan-subtask decomposition and scope.

A tracked Local Task may contain multiple independently deliverable plan subtasks. A plan subtask is a dispatch and execution unit within the tracked Local Task; do not create a new Local Task merely because a plan subtask is independently deliverable.

Each plan subtask must deliver one independent, verifiable outcome. Before dispatch, the controller must freeze a brief that states:

- the goal and observable result
- the scope boundary
- explicit non-goals
- the acceptance method: a focused command, assertion, reviewable artifact, or other concrete check

If a brief contains multiple outcomes that can be planned, implemented, verified, or delivered independently, split it into separate plan subtasks before dispatching. Do not expect a builder to infer, negotiate, or repair plan-subtask boundaries while implementing.

Before dispatch, set a finite execution budget for each plan subtask, such as a bounded change surface, validation effort, elapsed time, or a combination. A plan subtask that exceeds this budget is oversized even if it has one outcome; split it into independently verifiable intermediate plan subtasks. Do not expand the budget after dispatch.

## Core Workflow

1. Read the approved plan and apply the Plan-Subtask Sizing Gate to every builder handoff.
2. Check for contradictions, missing constraints, and unapproved decisions before starting.
3. Execute one frozen-scope plan subtask at a time with a fresh `builder` agent and its finite dispatch budget.
4. If `builder` returns `NEEDS_CONTEXT`, allow at most one bounded clarification retry. If it returns `BLOCKED`, or the dispatch budget is exhausted, stop the attempt and record it as blocked, deferred, or replaced by newly split plan subtasks. The controller decides whether to continue after it stops.
5. Review the completed plan subtask with `code-reviewer` using its scoped diff.
6. For Critical or plan-subtask-blocking Important findings, run at most one focused fix pass and one re-review.
7. If the re-review still has blocking findings, stop the patch loop; record the evidence and return to planning to split, re-estimate, or obtain a decision.
8. Mark a plan subtask complete only when its acceptance criteria are met; record non-blocking findings as separate follow-up work.
9. After all plan subtasks are done, run a broader final `code-reviewer` pass.
10. Record the final outcome in durable Local Task context. Do not close the tracked Local Task unless the user explicitly requests it.

For a tracked Local Task, the controller should use `context-task` to keep `plan.md`, `notes.md`, and the final outcome aligned with what the subagents actually discovered and completed.

## Controller Responsibilities

As the controller, keep your own context narrow too. Pass only what each agent needs:

- the plan subtask being worked on
- the relevant files or diff scope
- the constraints that bind that plan subtask
- the required verification steps

Do not paste large accumulated history into every dispatch.

For a tracked Local Task, the controller is also responsible for keeping the durable Local Task record coherent through `context-task` between handoffs rather than leaving progress only in agent responses.

## Scope Changes

Stop the current plan subtask when implementation or review reveals a need for a new architectural decision, a change to the frozen boundary, an unplanned dependency, or a change to the assumptions of later plan subtasks.

The controller must record the discovery in the durable Local Task context, then decide whether to continue. If it continues, update or rewrite the plan before continuing. Request user confirmation when the decision changes requested behavior, commitments, or tradeoffs that are not already approved. Do not append the work to the active plan subtask merely because it seems convenient or because the user asks to continue.

Stopping ends that plan-subtask attempt. Replanning must create new frozen plan-subtask briefs with their own execution budgets; do not redispatch the stopped plan subtask under expanded or rewritten scope.

## Handoff Contract For `builder`

When dispatching `builder`, include:

- plan-subtask name or number
- the frozen plan-subtask brief: goal, scope boundary, non-goals, acceptance method, and finite execution budget
- where the plan subtask fits in the plan
- any required interfaces or prior-plan-subtask outputs
- constraints that matter for this plan subtask
- verification commands or expected checks

Expect one of these statuses back:

- `DONE`
- `DONE_WITH_CONCERNS`
- `NEEDS_CONTEXT`
- `BLOCKED`

Treat any non-`DONE` status as a real signal, not noise.

## Handoff Contract For `code-reviewer`

When dispatching `code-reviewer`, include:

- the plan-subtask text or brief
- the intended behavior
- any binding constraints
- the changed files or diff range
- the `builder` summary of what changed and how it was verified

The review must answer two questions:

1. Did this plan subtask implement the right thing?
2. Is the result good enough to build on safely?

## Review Loop

If `code-reviewer` returns Critical or plan-subtask-blocking Important issues:

- send one focused fix pass through `builder`
- limit that pass to the reported findings within the frozen plan-subtask scope
- run one re-review after the fixes

A plan subtask gets no more than one fix-and-re-review cycle. If the re-review still has blocking findings, do not keep patching. Record the evidence in durable Local Task context, then return to planning to split or re-estimate the work, revise the plan, or request a required decision.

Findings that do not block the current acceptance criteria are separate follow-up work, not an expansion of the active plan subtask.

Do not continue to the next plan subtask with open plan-subtask-level issues that would make later work less reliable. If review findings change the plan-subtask understanding or remaining plan, update `notes.md` or `plan.md` through `context-task` before moving on.

## Final Review

After all plan subtasks are complete:

- dispatch `code-reviewer`
- review the broader change as a whole
- resolve serious findings before treating the branch as done

## Progress Tracking

Track progress outside transient conversation memory.

At minimum, maintain:

- which plan subtask is currently active
- which plan subtasks are complete
- which findings remain open

Use a file or durable Local Task tracking when the plan is long enough that session compaction or interruption is realistic. After a major milestone, a long-running operation, or a change in direction, restore the controller's working context from that durable record instead of accumulating session history.

For a tracked Local Task, use `context-task` to maintain `plan.md`, `notes.md`, and `outcome.md` as that durable record instead of ad hoc scratch notes.

## Observable Operations

Duration alone does not make an operation a problem. An operation needs special handling when it is resident or when the controller cannot observe a terminal result or defined progress signal within the current handoff.

For such operations, the controller must define before starting: the expected progress or health signal, the trigger for the next observation (such as a completion event, progress output, or a chosen periodic check), and the condition that ends waiting or requires escalation. Start the operation in the background when synchronous waiting would leave the agent unable to perform bounded work. Record each decisive result, unavailable-state outcome, or escalation in durable Local Task context before the next handoff. Do not leave an operation running with no observable state or decision point.

## Model Strategy

One advantage of dedicated agents is stable per-role model control.

Suggested defaults:

- `builder`: cheaper or mid-tier model for scoped implementation work
- `code-reviewer`: stronger reasoning model than `builder` for plan-subtask and whole-change review

Adjust upward when a plan subtask is unusually complex.

## Red Flags

Do not:

- run multiple builder plan subtasks in parallel against the same checkout
- dispatch a compound brief before splitting it with the Plan-Subtask Sizing Gate
- let a builder decide plan-subtask boundaries that the controller has not frozen
- append scope-changing work to an active plan subtask
- expand a plan subtask's execution budget or redispatch a stopped plan subtask under a rewritten scope
- retry `NEEDS_CONTEXT`, `BLOCKED`, or failed implementation work without a bounded dispatch budget
- skip scoped review between meaningful plan subtasks
- ignore `NEEDS_CONTEXT` or `BLOCKED`
- let controller context balloon with pasted diffs and old summaries instead of restoring durable context
- leave a resident or asynchronous operation running without an observable progress signal, next-observation trigger, and condition that ends waiting or requires escalation
- run more than one fix-and-re-review cycle without replanning
- move forward with unresolved plan-subtask-blocking Important or Critical review findings
- close a tracked Local Task automatically; only the user decides when to invoke `finishing-task`

## Bottom Line

This skill is about disciplined orchestration: one plan subtask, one fresh builder, one scoped review, then move on. A tracked Local Task may contain multiple such plan subtasks.

When the work is tracked, pair that orchestration with `context-task` so the durable Local Task documents reflect reality at each checkpoint.
