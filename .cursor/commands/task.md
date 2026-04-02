# /task — Decompose and Plan a Task

You are performing a structured decomposition of an engineering task for the current repository.

## Objective

Receive a task description, extract or generate a human-readable task ID, create a structured task folder under `tasks/<task-id>/`, and produce three artifacts: `plan.md`, `questions.md`, and `tracker.md`.

## Input

The user provides a task description in plain text:

```
{{input}}
```

## High-Level Requirements

### 1. Understand the Repository

Before planning, build context:

- Read `.cursor/resource/repo-explainer.md` if it exists — this is the primary source of repository architecture, patterns, and conventions.
- If no repo-explainer exists, quickly scan the repository structure, README, and key config files to understand the tech stack, architecture, and conventions.
- Identify the files, modules, and patterns relevant to the task.

### 2. Determine the Task ID

- **If a task ID is explicitly present** in the input (e.g., `JIRA-1234`, `GH-42`, or any recognizable identifier), use it as-is or incorporate it into the folder name.
- **If no task ID is present**, generate a human-readable slug:
  `task-<NN>-<short-kebab-description>`
  Examples: `task-01-add-auth-middleware`, `task-02-fix-search-pagination`, `task-03-refactor-user-service`.
  - Scan `tasks/` for existing `task-NN-*` folders and increment the number.
  - Keep the description to 3–5 words maximum.

### 3. Create the Task Folder

Create `tasks/<task-id>/` and populate it with the three files described below.

### 4. Generate `plan.md`

This is the implementation plan:

```markdown
# Task: <task-id>

## Summary
<!-- 2-3 sentences: what this task accomplishes and why it matters -->

## Scope
<!-- Which areas of the codebase are affected — modules, services, layers, etc. -->

## Implementation Steps
<!-- Numbered, ordered list of concrete changes. Each step must specify:
     - The file(s) to create or modify (full paths from repo root)
     - What changes are needed (new functions, modified logic, config updates, etc.)
     - Dependencies on other steps -->

### Step 1: <title>
**Files:** `path/to/file.ext`
**Changes:** ...
**Depends on:** None

### Step 2: <title>
...

## Testing Plan
<!-- What tests to write or update. What to verify manually. -->

## Dependencies
<!-- External dependencies: other teams, services, APIs, packages -->

## Rollback Strategy
<!-- How to undo these changes if something goes wrong -->
```

**Context awareness:** When analyzing the task:
- Reference the repository's established architecture and patterns from the repo-explainer.
- Respect existing conventions — naming, file organization, code style, error handling patterns.
- Reference existing similar code as examples for the developer to follow.

### 5. Generate `questions.md`

This captures every ambiguity, blocker, and implementation question:

```markdown
# Questions & Blockers: <task-id>

## Status Legend
- 🔴 **Blocker** — Cannot proceed without an answer
- 🟡 **Question** — Needs clarification but work can continue on other steps
- 🟢 **Resolved** — Answered

## Open Questions

### Q1: <question title>
**Status:** 🔴 Blocker | 🟡 Question
**Context:** Why this matters for the implementation
**Blocks:** Step X in plan.md
**Answer:** _Pending_

### Q2: ...
```

Proactively surface questions about:
- Ambiguous requirements or acceptance criteria
- Missing information about data models or APIs
- Cross-team or external service dependencies
- Edge cases and error handling expectations
- Performance, scalability, or security implications
- Environment-specific behavior (dev vs staging vs production)
- Backward compatibility concerns

If there are truly no questions, state that explicitly and explain why the task is unambiguous.

### 6. Generate `tracker.md`

This is a living document that tracks changes:

```markdown
# Change Tracker: <task-id>

## Summary
| Metric | Value |
|--------|-------|
| Files Created | 0 |
| Files Modified | 0 |
| Tests Added | 0 |
| Questions Resolved | 0/N |
| Status | 🟡 Planning |

## Change Log

<!-- Entries added by /execute as changes are made -->
<!-- Format:
### YYYY-MM-DD — <change title>
**Files:**
- `path/to/file` — Created | Modified | Deleted
**Details:** What was changed and why
**Related Step:** Step N from plan.md
-->

_No changes recorded yet. Use `/execute <task-id>` to begin implementation._
```

## Critical Constraints

1. **Never modify existing source files** during `/task`. This command is strictly for planning — zero writes outside the `tasks/<task-id>/` folder.
2. **Every step in `plan.md` must reference real file paths** that exist (or will exist) in the repository. Verify paths against the actual repo structure.
3. **Surface unknowns honestly.** If the task description is vague, do NOT fill gaps with assumptions. Move every assumption into `questions.md` as a 🟡 Question.

## Success Criteria

1. The folder `tasks/<task-id>/` exists with all three files fully populated.
2. `plan.md` contains concrete, actionable steps with real file paths that another engineer could follow without additional context.
3. Every ambiguity or external dependency is captured in `questions.md` with clear status indicators and blocking relationships to plan steps.
