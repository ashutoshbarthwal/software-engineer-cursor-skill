# /simplify — Create a Junior-Friendly Explanation of a Task

You are translating a senior-level implementation plan into plain language that any junior developer can follow — regardless of the tech stack or domain.

## Objective

Given a task ID, read its `plan.md` and produce a `plan-simplified.md` in the same task folder that explains every step in beginner-friendly language with concrete examples, visual aids, and zero assumed domain knowledge.

## Input

```
{{input}}
```

## High-Level Requirements

### 1. Load and Understand the Plan

- Read `tasks/<task-id>/plan.md` for the full implementation plan.
- Read `tasks/<task-id>/questions.md` to understand open blockers (mark blocked steps clearly).
- Read `.cursor/resource/repo-explainer.md` if it exists — for repository context, patterns, and terminology.
- If the task folder does not exist, inform the user and suggest running `/task` first.

### 2. Generate `plan-simplified.md`

Write `tasks/<task-id>/plan-simplified.md` using the structure below. The entire document must be written as if explaining to someone who:
- Has never seen this codebase before
- Knows basic programming but not the specific frameworks or tools used
- Does not know the domain-specific terminology without explanation
- Needs concrete examples, not abstract descriptions

### 3. Document Structure

```markdown
# <task-id> — Simplified Plan

## What Are We Doing? (The Big Picture)
<!-- 3-5 sentences in plain English. No jargon. Explain the business problem
     being solved, who benefits, and what the end result looks like.
     Use an analogy if it helps. -->

## Glossary (Terms You'll See)
<!-- A table of every acronym and domain term used in the plan.
     Each definition must be one sentence max. -->
| Term | What It Means |
|------|---------------|
| ...  | ... |

## How Things Connect (Before and After)
<!-- A diagram (ASCII or Mermaid) showing what changes.
     Label the "before" state and the "after" state.
     Keep it simple — boxes and arrows only. -->

## Step-by-Step Walkthrough

### Step <N>: <plain-english title>

**What we're doing:** <1-2 sentences, no jargon>

**Why:** <Why this step is needed — what breaks or is missing without it>

**Which files to touch:**
- `path/to/file` — <"Create this new file" | "Edit this existing file">

**What the code looks like:**
<!-- Show a SMALL, concrete before/after snippet or a template.
     Annotate every non-obvious line with a comment. -->

**How to verify it works:**
<!-- A concrete check the developer can run or look for.
     e.g., "Run the test suite and confirm the new test passes."
     e.g., "Start the app and visit /api/health to see the new field." -->

**Blocked?** <Yes — by Q<N> (brief reason) | No — ready to start>

<!-- Repeat for every step -->

## What's Blocked and Why
<!-- A plain-english summary of each blocker. -->

| Step | Blocked By | In Simple Terms | Who To Ask |
|------|-----------|-----------------|------------|
| ...  | ...       | ...             | ...        |

## Checklist (Copy-Paste This Into Your PR)
<!-- A flat checklist of every file that needs to be created or modified,
     in the order they should be done. -->
- [ ] Step 1: Edit `path/to/file.ext` — description
- [ ] Step 2: ...
```

### 4. Writing Rules

Follow these strictly:

1. **No unexplained acronyms.** Every acronym must appear in the Glossary and be spelled out on first use in the body.
2. **One idea per sentence.** Break compound sentences into two.
3. **Concrete over abstract.** Instead of "implement the middleware", write "create a function that runs before every API request to check if the user is logged in."
4. **Show, don't describe.** Prefer a 5-line code snippet over a paragraph of explanation. Annotate every line that isn't self-evident.
5. **Name the files.** Every step must list exact file paths. Never say "update the config" without saying which file.
6. **Explain the patterns.** When referencing repository patterns, explain what the pattern does and why, in one sentence each.
7. **Mark blocked steps visually.** Use a clear banner like `> **BLOCKED** — Cannot start until [Q6] is answered.`
8. **Include a verification step.** Every step must end with "How to verify it works" — a concrete action to confirm correctness.

## Critical Constraints

1. **Never alter `plan.md` or any other file.** This command only creates `plan-simplified.md`.
2. **Do not omit steps.** Every step in `plan.md` must appear in `plan-simplified.md`, even blocked ones. Blocked steps should be clearly marked but still explained.
3. **Do not add new steps.** The simplified plan must be a 1:1 translation of the original. If you notice something missing, note it under a "Potential Gaps" section.
4. **Keep code snippets short.** Show only the relevant fragment (5-15 lines). Use `// ... existing code ...` to indicate unchanged sections.

## Success Criteria

1. A developer with 6 months of experience can read `plan-simplified.md` end-to-end and understand what needs to be built, which files to touch, and in what order — without asking a senior engineer for clarification.
2. Every domain term, pattern, and acronym is explained in the Glossary or inline on first use.
3. Every step includes exact file paths, a before/after code example, and a concrete verification method.
