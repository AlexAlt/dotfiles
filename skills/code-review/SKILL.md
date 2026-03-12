---
name: code-review
description: Reviews branch changes for correctness, security, performance, edge cases, error handling, test quality, and completeness before pushing for human review. Use when the user asks for a code review, wants feedback on their changes, or is preparing to push a branch.
---

# Code Review

You are a thorough code reviewer. Your job is to review all changes on the current branch and provide structured, actionable feedback organized by severity. This review happens **before** pushing for human review.

## Workflow

### Step 1: Identify Changes

Run these commands to understand the full scope of changes:

```bash
# Find the base branch
git merge-base HEAD main

# Get the full diff against main
git diff main...HEAD

# List changed files
git diff main...HEAD --name-only

# Check for any uncommitted changes that should be included
git status
```

### Step 2: Read Changed Files

For each changed file, read the **full file** (not just the diff) to understand surrounding context. The diff alone is insufficient — you need to see how changes interact with existing code.

### Step 3: Review Against All Dimensions

Evaluate every change against each dimension below. Only report findings that are actionable — skip dimensions with no issues.

### Step 4: Produce the Review

Use the output format described below.

---

## Review Dimensions

### Correctness
- Does the logic produce the expected result?
- Are there off-by-one errors, nil/undefined dereferences, or incorrect conditionals?
- Do database queries return what the code expects?
- Are race conditions possible?

### Security
- SQL injection, XSS, mass assignment, or insecure deserialization?
- Are authorization checks present and correct?
- Is user input validated and sanitized?
- Are secrets, tokens, or PII exposed?

### Edge Cases
- What happens with empty collections, nil/null values, or zero-length strings?
- Boundary conditions: max/min values, empty states, concurrent access?
- Are default values sensible?

### Error Handling
- Are errors caught at appropriate levels?
- Do rescue/catch blocks handle the right exception types (not overly broad)?
- Are error messages meaningful for debugging?
- Can failures leave the system in an inconsistent state?

### Performance
- N+1 queries — are associations preloaded?
- Unnecessary iterations or redundant database calls?
- Missing database indexes for new queries/scopes?
- Large payloads or unbounded result sets?

### Test Quality
- Do tests cover the happy path AND failure cases?
- Are edge cases from the implementation tested?
- Do tests assert the right things (not just "doesn't crash")?
- Are mocks/stubs appropriate — not masking real behavior?
- Is test setup minimal and readable?

### Completeness
- Are all acceptance criteria addressed?
- Are migrations, serializers, routes, and permissions all updated consistently?
- Are feature flags wired up if needed?
- Is documentation updated where relevant?

### Backward Compatibility
- Do API changes break existing consumers?
- Are database migrations reversible and safe for zero-downtime deploy?
- Are removed/renamed columns still referenced elsewhere?
- Do interface/type changes break callers?

### Naming & Readability
- Are variable, method, and class names clear and consistent with codebase conventions?
- Is the code flow easy to follow without excessive comments?
- Are abstractions at the right level — not too clever, not too verbose?

### DRY & Design
- Is there duplicated logic that should be extracted?
- Do new abstractions follow existing project patterns (interactors, serializers, concerns)?
- Is code placed in the right layer (controller vs service vs model)?

### Type Safety
- TypeScript: are types explicit and precise (no `any`, no type assertions without justification)?
- Ruby: are method signatures clear with keyword args where appropriate?
- Do return types match what callers expect?

### Accessibility (UI changes only)
- Keyboard navigation supported?
- Screen reader friendly (ARIA labels, semantic HTML)?
- Color contrast and focus indicators?

### Observability
- Are important operations logged appropriately (following project logging guidelines)?
- Will errors be visible in monitoring?
- Are there breadcrumbs for debugging production issues?

---

## Output Format

```markdown
# Code Review: [branch name]

## Summary
[2-3 sentence overview: what the changes do, overall assessment, and biggest concern if any]

## Findings

### 🔴 Critical — Must fix before merge
[Issues that would cause bugs, security vulnerabilities, or data loss]

- **[Dimension]** `file:line` — Description of the issue and why it matters.
  - Suggested fix: [concrete suggestion]

### 🟡 Should Fix — Strongly recommended
[Issues that could cause problems or significantly hurt maintainability]

- **[Dimension]** `file:line` — Description and rationale.
  - Suggested fix: [concrete suggestion]

### 🟢 Nit — Optional improvements
[Style, naming, minor readability suggestions]

- **[Dimension]** `file:line` — Suggestion.

### ✅ Looks Good
[Briefly call out things done well — reinforces good patterns]

- [What was done well and where]
```

## Guidelines

- **Be specific**: always cite `file:line` for every finding.
- **Be actionable**: every finding should have a concrete fix or question.
- **Respect project conventions**: review against the project's style guides and rubocop/eslint config, not personal preference.
- **Don't nitpick formatting** that linters will catch automatically.
- **Prioritize**: a few critical findings are more valuable than dozens of nits.
- **Ask, don't assume**: if intent is unclear, frame it as a question rather than a demand.
