# /ask — Answer Questions About a Task or the Codebase

You are performing knowledge retrieval grounded in the task's planning artifacts and the broader repository context.

## Objective

Given a task ID and a free-form question (or just a question about the codebase), provide a precise, evidence-based answer — citing exact file paths, plan steps, question statuses, and code as evidence.

## Input

The user provides a task ID followed by a question, or just a question about the codebase:

```
{{input}}
```

## High-Level Requirements

### 1. Load Context

**If a task ID is provided:**
- Read `tasks/<task-id>/plan.md` — the implementation plan.
- Read `tasks/<task-id>/questions.md` — open/resolved questions and blockers.
- Read `tasks/<task-id>/tracker.md` — change log and progress.
- If `tasks/<task-id>/plan-simplified.md` exists, read it for additional context.
- If the task folder does not exist, inform the user and suggest running `/task` first.

**Always:**
- Read `.cursor/resource/repo-explainer.md` if it exists — for repository architecture and patterns.

### 2. Interpret the Question

Determine what the user is asking. Common categories:

| Category | What to look at |
|----------|----------------|
| **Status / Progress** — "What's done?", "What's left?" | `tracker.md` progress, summary metrics |
| **Blockers / Dependencies** — "What's blocking X?" | `questions.md` blocker items, `plan.md` dependencies |
| **Technical Detail** — "How does X work?" | `plan.md` details, actual source files |
| **Scope** — "Is X in scope?" | `plan.md` scope section |
| **Questions / Decisions** — "What's the answer to Q3?" | `questions.md` resolved answers |
| **File Locations** — "Where is X defined?" | Repo search, `tracker.md` change log |
| **Risk / Impact** — "What happens if X fails?" | `plan.md` rollback strategy |
| **How-to** — "How do I run tests?" | Repo docs, `repo-explainer.md` |

### 3. Answer from Evidence

Construct the answer using **only** information found in the task files and the repository:

1. **Direct answer** — Lead with the answer in 1-3 sentences.
2. **Evidence** — Cite the exact source(s):
   - For plan steps: "Per Step N in `plan.md`: ..."
   - For questions: "Per Q<N> in `questions.md` (status: 🟢/🟡/🔴): ..."
   - For tracker: "Per `tracker.md` entry YYYY-MM-DD: ..."
   - For code: "Per `path/to/file` line N: ..."
3. **Context** — If the answer has implications for other steps or decisions, note them.
4. **Gaps** — If the question cannot be fully answered, say what's missing and suggest how to get the answer (e.g., "Consider running `/unblock` to resolve Q3").

### 4. Support Follow-Up Patterns

If the user asks a follow-up in the same session, retain the task context — do not re-read files unless the task ID changes.

If the user's question reveals new information, suggest the appropriate command:
- `/unblock` to formally record an answer to an open question
- `/execute` if the answer unblocks implementation
- `/task` if the answer changes the implementation approach

### 5. Cross-Reference the Repository When Needed

If the question requires information beyond the task files:
- Read the referenced file directly from the repository.
- Cite the file path and relevant lines in the answer.
- Do NOT modify any files — `/ask` is read-only.

## Critical Constraints

1. **Read-only.** This command NEVER modifies any file. If a change is needed, recommend the appropriate command.
2. **Evidence-based answers only.** Every claim must trace back to a specific file and section. Do not speculate or fabricate details.
3. **Respect task boundaries.** Only answer questions about the specified task ID. If the question spans multiple tasks, answer for the specified one and note the dependency.
4. **No assumptions on open questions.** If the answer depends on an open 🟡/🔴 question, present the question's current status — do not fabricate a resolution.

## Success Criteria

1. The answer directly addresses the user's question with a clear, concise lead sentence.
2. Every factual claim cites its source file, section, or line number.
3. Gaps or uncertainties are explicitly flagged with a recommended next action.
