---
name: writing-plans
description: Use when you have approved requirements for multi-step work and need a concrete implementation plan before touching code.
---

# Writing Plans

Use this skill to turn requirements or an approved design into an implementation plan.

## Relationship To Context Skills

When the work has a tracked Local Task, use `context-task` alongside this skill.

- use `task_read` without a document to read the synthetic, read-only brief
- use `task_read` to read the relevant current `plan.md`, `notes.md`, and `outcome.md` context documents
- use `context-task` to maintain the context documents, including writing the resulting plan to `plan.md`

When prior project decisions or non-obvious history may matter, use `context-memory` before finalizing the plan.

## When to Use This Skill

Use this skill when:

- The work has multiple steps or checkpoints
- The user wants a plan before implementation
- The work spans several files or responsibilities
- The requirements are clear enough to sequence into plan subtasks

Do not use this skill for tiny, direct edits that can be implemented safely without separate planning.

## Core Principles

- Plan against the real codebase, not an imagined structure
- The controller owns plan-subtask scope; a plan must not silently enlarge it
- A tracked Local Task may contain multiple independently deliverable plan subtasks; do not create a new Local Task merely because a plan subtask is independently deliverable
- Freeze each plan subtask as an independently verifiable brief before execution
- Keep the plan concrete: exact files, responsibilities, and validation steps
- Prefer the smallest correct implementation
- Follow existing project patterns unless there is a strong reason not to

## Workflow

### 1. Confirm Inputs

Before writing the plan, make sure you have:

- The approved goal or requirements
- Enough codebase context to know where the change belongs
- A reasonable work boundary

If the work has a tracked Local Task, use `task_read` without a document to read its synthetic brief first. If research notes already exist, use `task_read` to read `notes.md` too.

If the request is too broad, split it into smaller plans. Within a tracked Local Task, split independently deliverable results into plan subtasks before execution; do not defer that split to a builder working on an active plan subtask. Do not create a new Local Task solely because a plan subtask is independently deliverable.

For every plan subtask, freeze a brief that states:

- **Goal:** the outcome this plan subtask alone must deliver
- **Scope boundary:** the allowed changes and interfaces
- **Non-goals:** related work that is explicitly excluded
- **Acceptance check:** an observable, independent verification of completion
- **Execution budget:** a finite limit appropriate to the work, such as a bounded number of steps, files, investigations, or attempts

The controller owns these briefs. Do not leave their scope open for an executor to reinterpret.

### 2. Map the File Structure

Before listing plan subtasks, identify:

- Which files will likely change
- Which new files may be needed
- What responsibility each file should have

Prefer focused responsibilities and clear interfaces. Avoid unnecessary restructuring.

If there may be relevant prior decisions, architecture notes, or durable discoveries, check `context-memory` before locking the file map and plan-subtask boundaries.

### 3. Break Work Into Plan Subtasks

Each plan subtask should:

- Produce one independently deliverable, meaningful completion boundary
- Have a frozen scope boundary and explicit non-goals
- Include the files involved
- Include an independent acceptance check
- Have a finite execution budget

Do not combine results merely because they are related. If either result can be delivered and verified independently, make it a separate plan subtask before execution. Keep those plan subtasks in the same tracked Local Task unless another reason requires separate Local Task tracking.

Good plan-subtask boundaries usually separate:

- setup or scaffolding that enables later work
- core behavior changes
- UI or integration wiring
- tests and validation
- documentation or follow-up cleanup when needed

### 4. Make Each Plan Subtask Concrete

For each plan subtask, include:

- Plan-subtask name
- Goal
- Scope boundary
- Non-goals
- Files to create or modify
- Main implementation steps
- Acceptance check and validation steps
- Finite execution budget
- Risks or dependencies if they matter

If execution reveals scope-changing work, do not add it to the active plan subtask. Stop that attempt. The controller decides whether to continue and, if so, must create a newly frozen brief for the additional or revised work. Do not repeatedly expand or retry a stopped plan subtask without a new brief.

Use exact file paths whenever you know them.

### 5. Include Verification

Every plan should say how to verify progress.

Examples:

- Run a focused test file
- Run the relevant lint command
- Exercise the feature manually
- Verify a specific regression case

### 6. Review the Plan in Isolation

After drafting the plan, send it to the `plan-reviewer` agent for a read-only review pass.

Give the reviewer:

- The plan path or plan content
- The original requirements or approved design
- Any project-wide constraints
- Any areas you are uncertain about

Use the review to catch:

- missing requirements
- oversized or poorly ordered plan subtasks
- vague steps or placeholders
- weak validation steps
- unnecessary scope

Fix the plan before presenting it as ready.

If the work has a tracked Local Task, use `context-task` to keep `plan.md` as the current source of planning truth rather than leaving the plan only in transient conversation history.

## Suggested Plan Format

```markdown
# <Feature Name> Implementation Plan

**Goal:** <one-sentence outcome>

**Context:** <key codebase or product constraints>

**Scope boundary:** <changes this plan may make>

**Non-goals:** <related work excluded from this plan>

## File Map

- Modify: `path/to/file.ts` - <responsibility>
- Create: `path/to/new-file.ts` - <responsibility>

## Plan Subtasks

### Plan Subtask 1: <name>

**Goal:** <what this plan subtask delivers>

**Scope boundary:** <allowed changes and interfaces>

**Non-goals:** <related work excluded from this plan subtask>

**Execution budget:** <finite limit on steps, files, investigations, or attempts>

**Files:**
- Modify: `path/to/file.ts`
- Create: `path/to/test.ts`

**Steps:**
1. <concrete action>
2. <concrete action>
3. <concrete action>

**Acceptance check / Verify:**
- Run: `<command>`
- Expect: <independently observable result>

### Plan Subtask 2: <name>
...
```

## Quality Bar

Do not write plans with placeholders like:

- TBD
- TODO
- implement later
- add tests
- handle edge cases

Replace vague instructions with explicit actions.

## Self-Review

After writing the plan, check:

1. Does every requirement map to a plan subtask?
2. Are plan-subtask boundaries small enough to verify and complete independently?
3. Are file paths and ownership clear?
4. Does every plan subtask have an explicit scope boundary, non-goals, acceptance check, and finite execution budget?
5. Are validation steps specific and independently verifiable?
6. Did you split independently deliverable results into plan subtasks before execution without creating a new Local Task solely for that reason?
7. Did you avoid unnecessary scope?

Fix obvious gaps inline before sending the plan to `plan-reviewer`.

After `plan-reviewer` returns findings:

1. Apply needed fixes to the plan
2. Re-check for consistency after the edits
3. Present the revised plan to the user

## Handoff

When the plan is complete:

- If the work has a tracked Local Task, use `context-task` to write or update `plan.md`
- Present the plan clearly to the user
- Mention that it has already passed through `plan-reviewer`
- Ask whether they want changes before implementation
- If approved, switch to implementation or the relevant execution workflow
