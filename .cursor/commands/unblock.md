# /unblock — Resolve Questions and Blockers Interactively

You are performing a structured triage of open questions and blockers for a planned task.

## Objective

Given a task ID, walk through every unresolved question and blocker in `questions.md`, present each one to the user for resolution, update the documents with answers, and record the resolution activity in `tracker.md`.

## Input

The user provides a task ID (and optionally, specific question numbers or answers inline):

```
{{input}}
```

## High-Level Requirements

### 1. Load the Task

- Read `tasks/<task-id>/questions.md` to get the full list of questions.
- Read `tasks/<task-id>/tracker.md` to understand current state.
- Read `tasks/<task-id>/plan.md` to understand which steps each question blocks.
- If the task folder does not exist, inform the user and suggest running `/task` first.

### 2. Identify Unresolved Items

Scan `questions.md` for all items with status 🔴 **Blocker** or 🟡 **Question** (anything that is NOT 🟢 **Resolved**).

Sort for presentation in priority order:
1. 🔴 **Blockers** first (these gate implementation progress)
2. 🟡 **Questions** second (these clarify but don't gate)

For each unresolved item, display:
```
---
**[Q<N>] <title>** — <🔴 Blocker | 🟡 Question>
Context: <1-2 sentence summary of why this matters>
Blocks: <which plan steps are gated>
---
```

### 3. Resolve Items One-by-One

Present unresolved items to the user and ask for answers. The user may:

- **Answer a specific question** — Update `questions.md` immediately:
  - Change status from 🔴/🟡 to 🟢 **Resolved**
  - Fill in the **Answer** field
  - Add the resolution date: `**Resolved:** YYYY-MM-DD`
  - If the answer has implementation implications, add an **Implications** note

- **Partially answer** — Provide new info but not a full resolution:
  - Keep the current status
  - Append new information to the **Answer** field with a date prefix

- **Downgrade a blocker** — No longer blocking:
  - Change status from 🔴 to 🟡 **Question**
  - Note why it was downgraded

- **Upgrade a question** — Actually blocking:
  - Change status from 🟡 to 🔴 **Blocker**
  - Add/update the **Blocks** field

- **Skip** — No answer yet. Move to the next item. No changes.

- **Add a new question** — Add it to `questions.md` with the next available Q number.

- **Answer multiple at once** — Process all of them.

### 4. Update `tracker.md` After Each Resolution

For every question that changes status, add a change log entry:

```markdown
### YYYY-MM-DD — Resolved Q<N>: <question title>
**Files:**
- `tasks/<task-id>/questions.md` — Modified
**Details:** <What was answered and any implementation implications>
**Impact:** <Which plan steps are now unblocked, if any>
```

Also update the summary table:
- Update `Questions Resolved` count (e.g., `5/10`)
- If ALL blockers are resolved, update Status to `🟢 Ready for Implementation`

### 5. Summarize at the End

After processing all items, provide:

```
## Unblock Summary

### Resolved This Session
- Q<N>: <title> — <brief answer>
- ...

### Still Open
- 🔴 Q<N>: <title> — <who needs to answer>
- 🟡 Q<N>: <title> — <who needs to answer>
- ...

### Newly Unblocked Steps
- Step <N>: <title> — was blocked by Q<X>, now ready
- ...

### Recommended Next Action
<What should the user do next — follow up with someone, run /execute, etc.>
```

## Critical Constraints

1. **Never assume answers.** If the user skips a question, leave it unchanged. Only record what the user explicitly states.
2. **Never modify `plan.md` during `/unblock`.** This command only touches `questions.md` and `tracker.md`. If an answer changes the plan, note it as an **Implication** and suggest updating the plan.
3. **Preserve question history.** When updating an answer, append rather than overwrite. Each update should be timestamped.
4. **Respect blocking relationships.** When a blocker is resolved, call out which plan steps are now unblocked.
5. **Atomic updates.** Update `questions.md` and `tracker.md` together — never leave them out of sync.

## Success Criteria

1. Every question the user answers is immediately reflected in `questions.md` with correct status, answer text, date, and implications.
2. `tracker.md` contains a timestamped entry for every status change, and summary metrics are accurate.
3. The session ends with a clear summary showing what was resolved, what remains, and the recommended next action.
