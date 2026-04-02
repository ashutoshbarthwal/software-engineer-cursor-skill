# Cursor AI Toolkit for Software Engineers

A portable `.cursor` setup that turns Cursor IDE into a structured engineering assistant. Built for junior developers but useful for anyone who wants AI-assisted planning, implementation, code review, and debugging — on any codebase, any tech stack.

## What's Included

```
.cursor/
├── commands/                  # Slash commands you invoke in Cursor chat
│   ├── explain-repo.md        # /explain-repo  — Generate a codebase explainer
│   ├── task.md                # /task          — Plan and decompose a task
│   ├── simplify.md            # /simplify      — Beginner-friendly plan translation
│   ├── ask.md                 # /ask           — Evidence-based Q&A
│   ├── unblock.md             # /unblock       — Resolve blockers interactively
│   ├── execute.md             # /execute       — Implement a plan step by step
│   ├── review.md              # /review        — Educational code review
│   └── debug.md               # /debug         — Structured debugging assistant
│
├── rules/
│   └── junior-dev.md          # Always-on AI behavior guidelines
│
├── skills/
│   └── code-documentation-code-explain/   # Skill for code analysis and explanation
│       ├── SKILL.md
│       └── resources/
│           └── implementation-playbook.md
│
└── resource/
    └── repo-explainer.md      # Generated — your project's living documentation
```

## Setup

### 1. Copy the `.cursor` folder into your project

```bash
# Clone this repo
git clone <this-repo-url> /tmp/cursor-toolkit

# Copy the .cursor folder into your project
cp -r /tmp/cursor-toolkit/.cursor /path/to/your/project/

# Clean up
rm -rf /tmp/cursor-toolkit
```

Or if you want to be selective:

```bash
# Copy only the commands and rules (minimal setup)
mkdir -p /path/to/your/project/.cursor/{commands,rules}
cp /tmp/cursor-toolkit/.cursor/commands/*.md /path/to/your/project/.cursor/commands/
cp /tmp/cursor-toolkit/.cursor/rules/*.md /path/to/your/project/.cursor/rules/
```

### 2. Open your project in Cursor

Open the project folder in Cursor IDE. The rules file (`.cursor/rules/junior-dev.md`) activates automatically on every file — no configuration needed.

### 3. Generate your repo explainer

Open Cursor chat and run:

```
/explain-repo
```

This analyzes your codebase and creates `.cursor/resource/repo-explainer.md` — a comprehensive onboarding document that all other commands reference. You only need to do this once (and re-run when the codebase changes significantly).

### 4. Add `tasks/` to your `.gitignore`

The `tasks/` directory is where the AI writes its analysis — generated plans, trackers, and question logs. You don't want to accidentally push this to your repo.

```bash
echo "tasks/" >> .gitignore
```

### 5. Start using the commands

You're ready. See the workflow section below.

## Commands

### `/explain-repo` — Understand the Codebase

Analyzes the entire repository and generates a living documentation file at `.cursor/resource/repo-explainer.md`. Covers architecture, data flow, design patterns, terminology, dev workflow, and common pitfalls.

**When to use:** First time on a project, or when the architecture has changed significantly.

```
/explain-repo
/explain-repo Focus on the authentication system
```

---

### `/task` — Plan Before You Code

Takes a task description and creates a structured folder at `tasks/<task-id>/` with three files:

| File | Purpose |
|------|---------|
| `plan.md` | Step-by-step implementation plan with file paths and dependencies |
| `questions.md` | Every ambiguity, blocker, and open question — tracked with status |
| `tracker.md` | Living change log updated as work progresses |

**When to use:** Before starting any non-trivial work.

```
/task Add rate limiting to the API endpoints
/task JIRA-1234 Migrate user service from REST to GraphQL
```

---

### `/simplify` — Make the Plan Beginner-Friendly

Takes a task ID and generates `plan-simplified.md` — the same plan rewritten with:
- Plain English (no unexplained jargon)
- A glossary of every technical term
- Before/after code snippets with line-by-line annotations
- A verification step for every change
- A copy-paste PR checklist

**When to use:** When a plan feels overwhelming or uses unfamiliar concepts.

```
/simplify task-01-add-rate-limiting
```

---

### `/ask` — Get Answers with Evidence

Ask anything about a task or the codebase. Every answer cites specific files, line numbers, and plan steps — no speculation.

**When to use:** Anytime you're confused or need to look something up.

```
/ask task-01 What steps are blocked right now?
/ask task-01 How does the existing auth middleware work?
/ask Where are the database migrations stored?
```

---

### `/unblock` — Resolve Blockers One by One

Walks through every open question and blocker in a task's `questions.md`. You can answer, skip, downgrade, upgrade, or add new questions. Every change is tracked with timestamps.

**When to use:** When you have answers to open questions, or need to review what's blocking progress.

```
/unblock task-01-add-rate-limiting
```

---

### `/execute` — Implement Step by Step

Reads the plan and implements each step in order. Checks for blockers first, tracks every file change in `tracker.md`, and follows existing code patterns.

**When to use:** When the plan is ready and blockers are resolved.

```
/execute task-01-add-rate-limiting
/execute task-01 Start from step 3
```

---

### `/review` — Learn from Code Review

Reviews your changes across four levels:

| Level | Meaning |
|-------|---------|
| 🔴 Critical | Bugs, security issues, data loss risks — must fix |
| 🟡 Important | Logic issues, missing error handling, performance — should fix |
| 🟢 Suggestion | Readability, naming, conventions — nice to have |
| 💡 Learning | Patterns, alternatives, deeper explanations — grow your skills |

Every finding explains *why* it matters and shows a concrete fix.

**When to use:** Before opening a PR, or when you want feedback on a file.

```
/review
/review src/api/auth.ts
/review task-01-add-rate-limiting
```

---

### `/debug` — Fix Bugs Systematically

Guides you through structured debugging: understand the problem, form hypotheses, investigate systematically, explain the root cause, apply the fix, and verify. Ends with a "what to watch for next time" learning note.

**When to use:** When something breaks and you're not sure why.

```
/debug TypeError: Cannot read property 'id' of undefined at UserService.getProfile
/debug The API returns 500 when I send an empty request body
/debug Tests pass locally but fail in CI
```

## How I Use It — The `plan/` + `tasks/` Workflow

This toolkit uses two directories to separate **your context** from the **AI's analysis**:

```
your-project/
├── plan/                     ← YOUR space — you create and edit this
│   ├── TASK-123/
│   │   ├── requirements.txt  # Whatever context you have about the task
│   │   ├── api-spec.pdf      # Sheet exports, PDFs, design docs
│   │   └── notes.md          # Your own notes, acceptance criteria, etc.
│   └── ASHU-456/
│       └── ticket-export.csv
│
├── tasks/                    ← AI's space — generated, don't touch
│   ├── TASK-123/
│   │   ├── plan.md           # AI-generated implementation plan
│   │   ├── questions.md      # Blockers and open questions
│   │   └── tracker.md        # Change log as work progresses
│   └── ASHU-456/
│       └── ...
│
└── .gitignore                ← tasks/ should be in here
```

### The flow

**1. You create a folder in `plan/` with your task ID and dump everything you have:**

```bash
mkdir -p plan/TASK-123
# Copy in whatever context you have — requirements, exports, screenshots, notes
```

**2. You run the `/task` command in Cursor, pointing it at your folder:**

```
/task @plan/TASK-123
```

The `@plan/TASK-123` tells Cursor to attach that folder's contents as context. The AI reads your materials, analyzes the codebase, and generates its structured plan in `tasks/TASK-123/`.

**3. From there, every other command works off `tasks/`:**

```
/simplify @tasks/TASK-123         ← Beginner-friendly version of the plan
/ask @tasks/TASK-123 <question>   ← Ask about the task
/unblock @tasks/TASK-123          ← Resolve open questions
/execute @tasks/TASK-123          ← Implement step by step
/review                           ← Review your changes before PR
/debug <error>                    ← When things break
```

**4. You keep track by checking `tasks/TASK-123/tracker.md`** — the AI updates it after every step.

### Why two directories?

| | `plan/` | `tasks/` |
|---|---------|---------|
| **Who writes** | You | The AI |
| **What's in it** | Raw context — PDFs, exports, notes, whatever you have | Structured analysis — plan, questions, tracker |
| **Should you edit it?** | Yes, it's yours | No, let the AI manage it |
| **Git** | Commit if you want to share context with your team | Add to `.gitignore` — it's generated |

## Recommended Workflow

```
/explain-repo                      ← Once, when you join the project

mkdir -p plan/TASK-123             ← Create your task folder, add your context
/task @plan/TASK-123               ← AI analyzes and creates the plan
/simplify @tasks/TASK-123          ← Get a beginner-friendly version
/ask @tasks/TASK-123 <question>    ← Clarify anything confusing
/unblock @tasks/TASK-123           ← Resolve open questions
/execute @tasks/TASK-123           ← Implement step by step
/review                            ← Review your changes before PR
/debug <error>                     ← When things break
```

## How the Rules Work

The file `.cursor/rules/junior-dev.md` is an always-on rule that shapes how the AI behaves across all interactions:

- **Explains the "why"** behind every change
- **Defines technical terms** on first use
- **Shows examples from your codebase**, not generic snippets
- **Matches existing code style** exactly
- **Asks before destructive operations**
- **Handles errors explicitly** — no swallowed exceptions

You can customize this file to match your team's conventions.

## Customization

### Add your own commands

Create a new `.md` file in `.cursor/commands/`. The filename becomes the slash command name. Use `{{input}}` to capture user input.

### Edit the rules

Modify `.cursor/rules/junior-dev.md` to add team-specific conventions, banned patterns, or preferred libraries.

### Update the repo explainer

Re-run `/explain-repo` anytime. If `.cursor/resource/repo-explainer.md` already exists, it will be updated rather than overwritten.

### Add more skills

Drop skill folders into `.cursor/skills/`. Each skill needs a `SKILL.md` file describing when and how to use it.

## FAQ

**Does this modify my source code?**
Only `/execute` modifies source files (it implements the planned changes). All other commands either create files in `tasks/` and `.cursor/`, or are read-only.

**Does this work with any language or framework?**
Yes. All commands are language-agnostic. They analyze whatever codebase they're placed in.

**Can I use this on an existing project?**
Yes. Copy the `.cursor` folder in and run `/explain-repo` to generate the initial documentation. Everything else works immediately.

**What if I don't want the task workflow?**
The commands are independent. Use `/review` and `/debug` on their own without ever touching `/task` or `/execute`.

**Should I commit the `.cursor` folder?**
Yes — commit it so the whole team gets the same commands and rules. Add `.cursor/resource/repo-explainer.md` to `.gitignore` if you prefer to generate it locally, or commit it as shared documentation.

**What should go in `.gitignore`?**
At minimum, add `tasks/` — it's AI-generated working files. Optionally add `.cursor/resource/repo-explainer.md` if you'd rather generate it locally. The `plan/` directory is up to you — commit it if you want to share task context with your team.
