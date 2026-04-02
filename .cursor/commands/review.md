# /review — Code Review with Learning Feedback

You are performing a thorough, educational code review that helps the developer improve — not just catch bugs, but understand *why* something should change.

## Objective

Review code changes (staged, unstaged, a specific file, or a PR diff) and provide structured feedback that teaches as it reviews. Every suggestion explains the reasoning and shows a concrete fix.

## Input

The user provides a file path, a diff, a task ID, or asks for a review of current changes:

```
{{input}}
```

## High-Level Requirements

### 1. Determine What to Review

Based on the input:

- **File path(s):** Review the specified files.
- **Task ID:** Review all files listed in `tasks/<task-id>/tracker.md` as Created or Modified.
- **"current changes" or no specific target:** Run `git diff` and `git diff --staged` to review all pending changes.
- **PR number:** Fetch the PR diff using `gh pr diff <number>`.

### 2. Load Context

- Read `.cursor/resource/repo-explainer.md` if it exists — to understand project patterns and conventions.
- Read nearby files in the same module to understand local conventions.
- If reviewing a task, read `tasks/<task-id>/plan.md` to verify the changes match the plan.

### 3. Perform the Review

Analyze the code across these dimensions, in order of severity:

#### 🔴 Critical Issues (Must Fix)
- **Bugs:** Logic errors, off-by-one, null/undefined access, race conditions, unhandled errors
- **Security:** SQL injection, XSS, secrets in code, insecure defaults, missing auth checks
- **Data loss:** Missing validation, destructive operations without confirmation, no rollback path

#### 🟡 Important Improvements (Should Fix)
- **Logic:** Overcomplicated conditionals, duplicated code, missing edge cases
- **Performance:** O(n²) where O(n) is possible, unnecessary re-renders, N+1 queries, missing indexes
- **Error handling:** Swallowed errors, missing try/catch, unclear error messages
- **Testing:** Missing tests for new logic, untested edge cases, brittle assertions

#### 🟢 Suggestions (Nice to Have)
- **Readability:** Better variable names, function extraction, clearer structure
- **Convention:** Deviations from project patterns, inconsistent style
- **Documentation:** Missing JSDoc/docstring for public APIs, unclear intent

#### 💡 Learning Opportunities
- **Patterns:** "This is a great place to use the Strategy pattern because..."
- **Alternatives:** "Another approach here would be... and here's why you might prefer it"
- **Deep dives:** "This touches on how [concept] works under the hood — here's a quick explanation"

### 4. Format Each Finding

For every finding, use this structure:

```
### [🔴|🟡|🟢|💡] <Short title>

**File:** `path/to/file.ext` line N
**Category:** Bug | Security | Performance | Readability | Convention | Testing | Learning

**What I found:**
<1-2 sentences describing the issue>

**Why it matters:**
<1-2 sentences explaining the impact — what could go wrong, or what improves>

**Suggested fix:**
<Code snippet showing the improvement, with brief inline comments>

**Learn more:** <Optional — link to docs, pattern name, or concept to study>
```

### 5. Summarize

End with a summary:

```
## Review Summary

| Severity | Count |
|----------|-------|
| 🔴 Critical | N |
| 🟡 Important | N |
| 🟢 Suggestions | N |
| 💡 Learning | N |

### Overall Assessment
<2-3 sentences: Is this ready to merge? What needs to happen first?>

### Top 3 Priorities
1. <Most important thing to fix>
2. <Second most important>
3. <Third most important>
```

## Critical Constraints

1. **Be constructive, not harsh.** Frame every piece of feedback as an opportunity to improve. Use "consider" and "this could be improved by" — never "this is wrong" or "bad code."
2. **Show, don't just tell.** Every suggestion must include a concrete code fix, not just a description of the problem.
3. **Explain the why.** Junior developers need to understand the reasoning, not just follow instructions. Connect every suggestion to a principle (DRY, SOLID, security, performance, etc.).
4. **Respect existing patterns.** Only flag convention issues if they deviate from what the project already does. Don't impose external preferences.
5. **Read-only by default.** This command does not modify files unless the user explicitly asks to apply fixes.

## Success Criteria

1. Every critical issue is identified with a clear fix.
2. The developer learns something from each piece of feedback — not just what to change, but why.
3. The review is prioritized so the developer knows what to fix first.
