---
description: Guidelines for AI-assisted development — optimized for learning and safety
globs: **/*
---

# Junior Developer Guidelines

## Communication Style

- Explain the "why" behind every change, not just the "what."
- Define technical terms on first use. Never assume domain knowledge.
- When suggesting a pattern, show a concrete example from this codebase — not a generic textbook snippet.
- Prefer short, annotated code blocks over long prose explanations.
- When multiple approaches exist, briefly name them and explain why you're choosing one.

## Before Making Changes

- Always read the file before editing it. Understand the surrounding code.
- Check `.cursor/resource/repo-explainer.md` for project architecture, patterns, and conventions.
- Look at nearby files in the same module for style, naming, and structural conventions.
- If a `tasks/` folder exists for the current work, check `plan.md` for context on what's being built and why.

## Code Quality

- Match the existing code style exactly — indentation, quotes, naming conventions, import order.
- Never introduce a new pattern when an existing one covers the use case. Consistency beats novelty.
- Every new function or module should have a clear, single responsibility.
- Handle errors explicitly. No empty catch blocks. No swallowed exceptions.
- Add tests for new logic. Follow the existing test patterns in the project.

## Safety

- Never commit secrets, API keys, or credentials. Check for `.env` files before staging.
- No destructive operations (deleting data, dropping tables, force-pushing) without explicit confirmation.
- When unsure about a change's impact, ask before proceeding. It's better to ask than to break something.
- Keep changes small and focused. One PR should do one thing.

## Learning Workflow

These commands are available to help you work through tasks:

1. **`/explain-repo`** — Generate a comprehensive explainer for the codebase. Run this first when joining a new project.
2. **`/task <description>`** — Break down a task into a structured plan with steps, questions, and tracking.
3. **`/simplify <task-id>`** — Translate the plan into beginner-friendly language with examples and a glossary.
4. **`/ask <task-id> <question>`** — Ask questions about a task or the codebase. Get evidence-based answers.
5. **`/unblock <task-id>`** — Walk through open questions and blockers interactively.
6. **`/execute <task-id>`** — Implement the planned changes step by step.
7. **`/review`** — Get an educational code review with explanations and learning notes.
8. **`/debug <description>`** — Debug an issue through systematic investigation.

## Recommended Workflow for New Tasks

```
/explain-repo          ← Do this once when you join the project
/task <description>    ← Plan the work
/simplify <task-id>    ← Get a beginner-friendly version
/ask <task-id> <?>     ← Clarify anything confusing
/unblock <task-id>     ← Resolve any blockers
/execute <task-id>     ← Implement step by step
/review                ← Review your changes before PR
```
