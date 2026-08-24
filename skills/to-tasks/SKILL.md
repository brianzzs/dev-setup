---
name: to-tasks
description: Break a local plan created by grill-me or a spec created by to-spec into linked, trackable Markdown task files. Use when the user asks to split an approved plan or spec into implementation tasks.
disable-model-invocation: true
---

# to-tasks

Turn one existing local plan or spec into a set of small or medium implementation tasks. Keep the source document as the main story and task tracker. Create one Markdown file for each task.

This skill plans work only. Do not implement the feature, change production code, create commits, or publish anything.

## Find the source document

1. Find the repository root and inspect the working tree.
2. Use a path supplied with the command when present.
3. Otherwise, use the plan or spec identified in the current conversation.
4. If the conversation does not identify it, search these locations:
   - `.pi/plans/`
   - `.pi/specs/`
   - `.scratch/`
   - `specs/`
   - `docs/specs/`
5. Prefer the document created or updated by the most recent `grill-me` or `to-spec` run in the conversation. Do not select a document only because its file modification time is newest.
6. If more than one document is plausible and the conversation does not resolve the choice, ask the user to select one. Do not merge separate plans.
7. Read the full source document before writing task files. Also inspect repository instructions and enough relevant code, tests, and project metadata to make each task concrete.

The source document remains the authority. Do not invent requirements or reverse confirmed decisions. Preserve its acceptance criteria, scope limits, terminology, and unresolved questions.

## Choose task file locations

Create task files beside the source document in a directory named `<source-slug>.tasks/`.

Example:

```text
.pi/specs/export-audit-log.md
.pi/specs/export-audit-log.tasks/
  01-add-export-query.md
  02-expose-export-endpoint.md
  03-cover-export-workflow.md
```

Use two-digit sequence numbers and short lowercase kebab-case slugs. Keep sequence numbers stable when updating an existing breakdown. Add new tasks after existing tasks unless the dependency order would become misleading.

Keep the task files local and untracked when the source is local and untracked. In a Git repository, check with `git check-ignore`. If needed, add only the task directory path to `.git/info/exclude`. Never edit the tracked `.gitignore` for these local planning files. Preserve all unrelated user changes.

## Set task boundaries

A task must have one clear outcome that an implementation agent can complete and verify in one focused work session.

A good task:

- Covers one behavior, boundary, migration step, or testable integration.
- Names the likely code area without prescribing code that repository inspection does not support.
- Has observable acceptance criteria.
- Includes the tests and validation needed for its own change.
- States dependencies on other tasks.
- Can be reviewed on its own without unrelated cleanup.

Do not create one task per file or one task per minor edit. Keep tightly coupled code and its focused tests in the same task. Split work when it crosses independent behaviors, architectural boundaries, migrations, or test seams.

Order tasks by dependency. Prefer vertical slices that leave the repository in a valid state. Do not create a separate catch-all testing task when focused tests belong with each behavior. A final integration or regression task is valid only when it proves behavior that the earlier task seams cannot prove.

Do not repeat the full source document in every task. Link to it and copy only the decisions and acceptance outcomes needed to execute that task without ambiguity.

## Handle work that is not ready

Do not disguise an epic as a task. A task needs more breakdown when it has several independent outcomes, spans many unrelated areas, or cannot be verified with one coherent validation set.

If confirmed source material is not sufficient to split an item safely:

1. Create a task file for the unresolved item so it remains tracked.
2. Set its status to `blocked: needs grilling` or `blocked: needs breakdown`.
3. State the exact decisions, research, or dependency needed before implementation.
4. Do not invent subtasks behind the unresolved decision.
5. Add it to the source task list with its blocked status.
6. List all such items at the end of the command output.

Use `needs grilling` when user or product decisions are missing. Recommend another `grill-me` pass for that item. Use `needs breakdown` when the requirements are settled but the work still needs repository research or technical decomposition.

## Write each task

Use this template:

```markdown
# <Task title>

Status: pending
Parent: [<source title>](../<source-file-name>.md)
Depends on: None

## Outcome

One concrete result this task must produce.

## Scope

- Work included in this task.
- Relevant boundaries or likely code areas.

## Out of scope

- Work reserved for another task or excluded by the source plan.

## Requirements

- Confirmed decisions and behavior from the parent document that apply here.

## Acceptance criteria

- [ ] Observable result that proves the outcome.

## Tests and validation

- Focused behavior to test and the highest useful test seam.
- Repository commands to run when known.

## Implementation notes

Repository facts, prior art, risks, or sequencing details. Do not add speculative requirements.

## Completion record

- Not started.
```

For blocked tasks, replace `Status: pending` with the applicable blocked status. In `Implementation notes`, add a `### Needed before implementation` subsection with the unresolved items.

## Update the source tracker

Add or update a `## Tasks` section in the source document. Place it near the source document's work-tracking section when one exists. Otherwise, place it before final notes or append it at the end.

Use a linked checklist in dependency order:

```markdown
## Tasks

- [ ] [01 Add export query](export-audit-log.tasks/01-add-export-query.md), pending
- [ ] [02 Expose export endpoint](export-audit-log.tasks/02-expose-export-endpoint.md), depends on 01
- [ ] [03 Resolve retention behavior](export-audit-log.tasks/03-resolve-retention-behavior.md), blocked: needs grilling
```

The checkbox is the source of truth for completion at story level. Keep the text status aligned with the task file. Do not mark a task complete unless its acceptance criteria are complete and its listed validation ran successfully.

If the source has a generic `Work to do` list, replace duplicate implementation bullets with a short pointer to `## Tasks`. Preserve acceptance checklists, work logs, validation history, decisions, and notes.

Make reruns idempotent:

- Reuse matching task files and links.
- Preserve completed checkboxes, completion records, validation results, and user notes.
- Update a task only when the source document changed or the existing breakdown is incomplete.
- Do not renumber completed tasks.
- Record material breakdown changes in the source change log when it has one.

## Check the breakdown

Before finishing:

1. Map every in-scope source acceptance outcome to at least one task.
2. Check that no task adds behavior excluded by the source.
3. Check dependency links and relative paths.
4. Check that every task has an outcome, scope, acceptance criteria, and validation plan.
5. Check that each task is small or medium. Mark oversized or unresolved work as blocked instead of pretending it is ready.
6. Inspect the final diff. Confirm that only the source plan, its task files, and any required local Git exclude entry changed.

## Finish

Report:

- The repository-relative path of the source document.
- The task directory and number of task files created or updated.
- The ordered task titles and statuses.
- Any assumptions.
- A final `Needs more breakdown or grilling` section. List blocked items and why. Write `None` when all tasks are ready.

Do not claim that implementation or implementation validation is complete.
