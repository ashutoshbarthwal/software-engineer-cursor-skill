# /explain-repo — Generate a Repository Explainer Document

You are generating a comprehensive repository explainer that helps any developer — especially those new to the codebase — understand the project's purpose, architecture, patterns, and conventions.

## Objective

Analyze the entire repository and produce a `repo-explainer.md` document in `.cursor/resource/` that serves as a living onboarding guide. This document becomes the shared context that other commands (`/task`, `/simplify`, `/execute`, etc.) reference when working in this codebase.

## Input

Optional — the user may provide focus areas or specific questions:

```
{{input}}
```

If no input is provided, perform a full repository analysis.

## Skill Reference

Read and follow the skill at `.cursor/skills/code-documentation-code-explain/SKILL.md` for explanation techniques, visual aids, and progressive disclosure patterns. Use `resources/implementation-playbook.md` for detailed templates when needed.

## Instructions

### 1. Discover the Repository

Systematically explore the codebase to understand its shape:

- **Root files:** Read `README.md`, `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `Makefile`, `docker-compose.yml`, or whatever dependency/config files exist at the root to identify the language, framework, and tooling.
- **Directory structure:** Map the top-level and second-level directory tree. Identify which folders contain source code, tests, config, docs, CI/CD, and infrastructure.
- **Entry points:** Find the main entry point(s) — `main.py`, `index.ts`, `cmd/`, `src/main.rs`, etc.
- **Configuration:** Read CI/CD configs (`.github/workflows/`, `Jenkinsfile`, `.gitlab-ci.yml`), linter configs, and environment files to understand the dev workflow.
- **Documentation:** Read any existing docs to avoid duplicating effort and to cross-reference accuracy.

### 2. Analyze Architecture and Patterns

Go deeper into the source code:

- **High-level architecture:** Identify the architectural style (monolith, microservices, serverless, layered, event-driven, etc.) and the major components/modules.
- **Data flow:** Trace how data enters the system, gets transformed, and exits. Identify databases, message queues, APIs, and external integrations.
- **Key abstractions:** Find the core domain models, services, controllers, and their relationships.
- **Design patterns:** Identify recurring patterns (repository pattern, middleware chains, pub/sub, CQRS, etc.).
- **Error handling:** How does the codebase handle errors, logging, and observability?
- **Testing strategy:** What testing frameworks are used? Unit, integration, e2e? How are tests organized?

### 3. Write the Repo Explainer

Create `.cursor/resource/repo-explainer.md` with this structure:

```markdown
# <Project Name> — Repository Explainer

## What This Repository Does
<!-- 2-3 sentences in plain English. What problem does it solve? Who uses it? -->

---

## High-Level Architecture
<!-- ASCII or Mermaid diagram showing major components and data flow.
     Label every box and arrow. Keep it to one screen. -->

---

## Repository Structure
<!-- Directory tree with annotations explaining what each major folder contains.
     Go 2-3 levels deep for important areas, 1 level for the rest. -->

---

## How It Works — End to End
<!-- Trace one complete user action or data flow through the system.
     Pick the most representative example. Show which files are involved
     at each stage. -->

---

## Key Design Patterns
<!-- For each pattern used in the codebase:
     1. Name the pattern
     2. Explain what it does in one sentence
     3. Show a concrete example from this repo (file path + snippet)
     4. Explain why this pattern was chosen -->

---

## Data Model
<!-- Describe the core entities/models and their relationships.
     Use a diagram if there are more than 3 entities.
     Include database schema details if relevant. -->

---

## Key Concepts & Terminology
<!-- Table of domain-specific terms, abbreviations, and internal jargon.
     | Term | Meaning |
     |------|---------|  -->

---

## Development Workflow
<!-- How to set up the dev environment, run the app, run tests,
     and deploy. Step-by-step for a new developer. -->

---

## Testing Strategy
<!-- What types of tests exist, how they're organized,
     how to run them, and what the coverage expectations are. -->

---

## Deployment & CI/CD
<!-- How code gets from a developer's machine to production.
     Include environment descriptions if multiple exist. -->

---

## Pitfalls & Edge Cases
<!-- Things that have bitten developers before. Non-obvious behaviors.
     Environment-specific gotchas. Common mistakes. -->

---

## Suggested Next Steps for New Developers
<!-- Ordered list of what to read, study, and try first.
     Reference specific files and commands. -->
```

### 4. Validate the Document

After writing the explainer:

- Verify every file path mentioned actually exists in the repo.
- Ensure no section is left empty — if information isn't available, say so explicitly rather than leaving a blank.
- Check that diagrams are syntactically correct (Mermaid or ASCII).
- Confirm the document is self-contained — a new developer should not need to read any other file to understand the repository at a high level.

## Critical Constraints

1. **Ground everything in the actual code.** Every claim must be verifiable by reading a specific file. Do not invent architecture, patterns, or conventions that don't exist in the repo.
2. **Do not modify any source code.** This command only creates or updates `.cursor/resource/repo-explainer.md`.
3. **Keep it current.** If a `repo-explainer.md` already exists, read it first and update it rather than starting from scratch. Note what changed at the top.
4. **Explain, don't assume.** Write for someone who has never seen this codebase, this framework, or this domain. Define every acronym. Explain every pattern.
5. **Prefer diagrams over paragraphs.** A good ASCII diagram replaces 500 words of description.

## Success Criteria

1. A developer joining the team can read `repo-explainer.md` in 15-20 minutes and have a working mental model of the codebase — enough to navigate, find things, and start making small changes.
2. Every file path, command, and pattern referenced in the document is accurate and verifiable.
3. The document covers architecture, data flow, patterns, development workflow, and common pitfalls — no critical blind spots.
