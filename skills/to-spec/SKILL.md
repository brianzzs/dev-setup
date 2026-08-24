---
name: to-spec
description: Turn the current conversation and repository context into a local implementation spec. Create an untracked Markdown file for the agent to follow and update.
disable-model-invocation: true
---

# to-spec

Turn the current conversation into a practical implementation spec. Use the repository to fill in technical context, then save the spec inside the repository for the implementation agent to use.

This skill is local only. Do not publish the spec, call an external service, apply labels, or create a commit.

## Rules

- Do not interview the user. Do not ask for confirmation of the plan, test seams, or wording.
- Use the current conversation as the product source of truth.
- Use the repository as the technical source of truth.
- Do not invent requirements. When the context is incomplete, record a clear assumption or unresolved question in `Further Notes` and continue.
- Preserve the repository's terminology, domain glossary, architecture, and ADRs.
- Do not implement the feature while running this skill. The only intended repository change is the local spec file and, when needed, a local Git exclude entry that keeps it untracked.
- Preserve existing user changes. Do not reset, clean, stage, or commit anything.

## Choose the spec file

1. Find the repository root and inspect the working tree before making changes.
2. Look for an existing local spec for the feature. Check these locations in this order when they exist:
   - `.pi/specs/`
   - `.scratch/`
   - `specs/`
   - `docs/specs/`
3. Reuse a matching existing spec instead of creating a duplicate. Preserve its work log, completed items, and useful notes.
4. If no matching spec exists, create `.pi/specs/<feature-slug>.md`. Use a short, lowercase, kebab-case slug.
5. Keep the spec out of commits:
   - Check whether the file is ignored with `git check-ignore`.
   - If it is not ignored, add the specific path or directory to `.git/info/exclude`, not to the project's tracked `.gitignore`.
   - If the existing candidate is tracked, do not change it into a working spec. Create a new ignored file instead.
   - If the repository is not a Git repository, create the file in the chosen local spec directory and state that it must remain uncommitted.
6. Do not overwrite unrelated content in an existing file. Update the relevant sections in place.

The spec is a working document, not a deliverable to commit. An implementation agent must read it before changing code and update its work-tracking sections as the work progresses.

## Explore the repository

Before drafting the spec, inspect enough of the repository to understand the change:

- Repository instructions, such as `AGENTS.md`, `CONTRIBUTING.md`, and local developer documentation.
- The project metadata, build commands, test commands, and validation tools.
- The domain glossary, if the project has one.
- ADRs and design documents that apply to the affected area.
- Relevant modules, callers, existing behavior, and nearby tests.
- The current Git status and any existing diff that may affect the feature.

Do not broaden the feature because of unrelated code you find. Use current behavior and established patterns to explain the smallest coherent change.

## Select test seams

Identify the smallest number of seams at which the feature can be tested through observable behavior.

- Prefer an existing seam over a new test hook.
- Use the highest seam that gives reliable, focused feedback, such as an endpoint, command, user-visible workflow, or other public boundary.
- Avoid testing private helpers or implementation details when a higher seam can prove the behavior.
- If a new seam is necessary, propose it at the highest useful level and explain why it is needed.
- Prefer one seam. Add more only when separate boundaries are required by the behavior.
- Record the chosen seam, the behavior it proves, and relevant prior art in `Testing Decisions`.

Do not ask the user whether the seams are acceptable. Make the best decision from the conversation and codebase, and record any uncertainty as an assumption.

## Write the spec

Use the following template. Replace every placeholder with information from the conversation and repository. Keep the original section names. Add only details that help an implementation agent make or verify the change.

```markdown
# <Feature title>

Status: ready for implementation
Last updated: <date>

## Problem Statement

The problem the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A thorough, numbered list of user stories. Use this format:

1. As an <actor>, I want a <feature>, so that <benefit>

Cover the relevant actors and observable behavior, including normal use, validation, failure cases, permissions, empty states, compatibility, and recovery. Include only stories supported by the conversation or repository. Do not pad the list with invented edge cases.

## Implementation Decisions

A list of decisions that an implementation agent must follow. Include, when relevant:

- Modules or responsibilities to build or modify
- Interfaces and contracts to change
- Technical clarifications from the conversation
- Architectural decisions and constraints
- Schema or data changes
- API behavior and error contracts
- Specific user interactions
- Compatibility and migration rules

Do not include specific file paths or code snippets. A short prototype snippet is allowed only when it records a decision more precisely than prose, such as a state machine, reducer, schema, or type shape. Include only the decision-rich part and say that it came from a prototype.

Separate confirmed decisions from assumptions. Do not turn guesses into requirements.

## Testing Decisions

Describe:

- What a good test proves through external behavior, rather than implementation details
- The highest test seam or seams selected for the feature and why
- Which modules or boundaries need coverage
- Success, validation, authorization, not-found, conflict, and failure cases that apply
- Similar tests in the repository that provide useful prior art
- Any test seam that must be added, with the reason it cannot use an existing seam

## Out of Scope

Describe what this spec does not cover. Include tempting adjacent work that an implementation agent must not add.

## Work tracking

### Acceptance checklist

- [ ] <observable acceptance outcome>

### Work to do

- [ ] <implementation task>
- [ ] <test or validation task>

### Work done

- None yet.

### Validation

- Not run. This skill creates the spec; implementation validation belongs to the implementation work.

### Change log

- <date>: Created or updated from the current conversation and repository state.

## Further Notes

Record assumptions, unresolved questions, risks, dependencies, and any repository facts that an implementation agent needs. Do not use this section to ask the user a question. If the conversation does not resolve a detail, state the safest interpretation and the consequence of being wrong.
```

## Keep the spec useful during implementation

When an implementation agent uses the file:

1. Read the full spec before editing code.
2. Treat the confirmed decisions and out-of-scope list as the working contract.
3. Convert the implementation decisions and acceptance outcomes into concrete items under `Work to do`.
4. Mark an item complete only after the related work is actually done.
5. Add concise entries under `Work done` as meaningful work finishes.
6. Record the exact validation commands and their results under `Validation`. Do not claim a check passed unless it ran.
7. If new information changes the plan, update the relevant decision, acceptance item, or further note before continuing. Add the reason to `Change log`.
8. Set `Status` to `in progress`, `blocked`, or `complete` as appropriate. Use `complete` only when the implementation, focused tests, required broader checks, and final diff review are done.
9. Leave the spec untracked. Never stage or commit it as part of the feature.

## Finish

After writing the file, report its repository-relative path, current status, and any assumptions or unresolved risks. Do not ask the user to approve the spec, and do not claim that the feature is implemented.
