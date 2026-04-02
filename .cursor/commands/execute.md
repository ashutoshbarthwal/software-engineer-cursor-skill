# /execute — Implement a Planned Task

You are systematically implementing a previously planned engineering task from its `tasks/<task-id>/` folder.

## Objective

Given a task ID, read the task's plan, resolve any open blockers, and implement the changes step by step — updating the tracker and questions documents as work progresses.

## Input

The user provides a task ID (and optionally, answers to open questions or a specific step to start from):

```
{{input}}
```

## High-Level Requirements

### 1. Load and Validate the Task

- Read `tasks/<task-id>/plan.md` to understand the full implementation plan.
- Read `tasks/<task-id>/questions.md` to check for open blockers and questions.
- Read `tasks/<task-id>/tracker.md` to see what has already been done.
- Read `.cursor/resource/repo-explainer.md` if it exists — for repository patterns and conventions.
- If the task folder does not exist, inform the user and suggest running `/task` first.

### 2. Resolve Blockers Before Proceeding

**If `questions.md` contains any 🔴 Blocker items:**
- Present each unresolved blocker clearly, explaining what decision is needed and what plan steps are blocked.
- Wait for the user's response.
- Once answered, update `questions.md`:
  - Change status from 🔴 to 🟢 Resolved.
  - Fill in the **Answer** field with the user's response and the date.
  - Add any implementation implications.

**If `questions.md` contains 🟡 Question items:**
- Present them but do NOT block execution. Note any assumptions and proceed.
- If the user provides answers, update `questions.md` accordingly.

**If all items are 🟢 Resolved or there are no questions:**
- Proceed directly to implementation.

### 3. Implement Changes Systematically

Execute the steps from `plan.md` **in order**, respecting dependencies.

For each step:

1. **Announce** which step you are implementing and what files will be touched.
2. **Read existing files** before modifying them — never edit blind.
3. **Make the changes** following the repository's established patterns:
   - Study nearby code for style, naming, error handling, and structural conventions.
   - Match the existing code style exactly — indentation, quotes, imports, exports.
   - Follow the project's testing conventions when adding tests.
   - Respect linter and formatter configurations.
4. **Update `tracker.md`** immediately after each step completes:
   - Add a change log entry with date, files touched, and what was done.
   - Update the summary metrics (Files Created, Files Modified, etc.).
   - Mark the related plan step as done.
5. **Check for lint errors** after making changes and fix any you introduced.

### 4. Handle Discovered Issues During Implementation

If during implementation you discover:

- **A new question or ambiguity:** Add it to `questions.md` as 🟡 Question, note it in the tracker, and continue with other steps if possible.
- **A blocker that prevents further progress:** Add it to `questions.md` as 🔴 Blocker, update the tracker status, and pause to ask the user.
- **A plan step that needs revision:** Note the discrepancy in the tracker and propose the adjustment to the user before proceeding.
- **A dependency on an earlier step that wasn't completed:** Complete the dependency first, then proceed.

### 5. Finalize

After all steps are complete (or all non-blocked steps are done):

1. Update `tracker.md` summary:
   - Set Status to 🟢 Complete (or 🟡 Partially Complete if blockers remain).
   - Update all metric counts.
2. If any 🟡 Questions were assumed, list the assumptions made in the tracker.
3. Summarize to the user what was done, what remains, and any follow-up actions needed.

## Critical Constraints

1. **Respect the plan order.** Do not skip ahead or reorder steps unless a dependency forces it. Document deviations in the tracker.
2. **Never silently skip a blocker.** Every 🔴 Blocker must be resolved by the user before its dependent steps are executed.
3. **Every file change must be tracked.** If you touch a file, it MUST appear in `tracker.md`. Zero untracked changes.
4. **Preserve existing patterns.** Do not introduce new conventions that diverge from what exists in the repo. When in doubt, find the closest existing example and follow it exactly.
5. **No destructive operations without confirmation.** If a step requires deleting files, dropping data, or making breaking changes, confirm with the user first.

## Success Criteria

1. All non-blocked steps from `plan.md` are implemented with zero deviation from established repository patterns.
2. `tracker.md` accurately reflects every file created or modified — an external reviewer could reconstruct the full change set from the tracker alone.
3. `questions.md` is fully up-to-date: answered questions marked 🟢, new discoveries added with correct severity, and no silent assumptions on blockers.
