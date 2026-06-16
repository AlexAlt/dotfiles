---
name: code-review
description: Reviews branch changes with emphasis on architecture, complexity, and readability — plus correctness, security, performance, edge cases, error handling, tests, and completeness — before pushing for human review. Use when the user asks for a code review, wants feedback on their changes, or is preparing to push a branch.
---

# Code Review

Review all changes on the current branch and give structured, actionable feedback by severity, **before** pushing for human review.

The primary lens is **design quality**: prefer the simplest organization that satisfies the requirements. Flag complexity that isn't paying for itself — premature abstractions, unnecessary layers, defensive code for states that can't occur, and "flexible" generality nothing uses. Adding code is a cost; deleting it is often the better fix.

## Workflow

1. **Identify changes**: `git diff main...HEAD` (use `git merge-base HEAD main` to confirm the base), `--name-only` for the file list, and `git status` for uncommitted work.
2. **Read full files**, not just diffs — you need surrounding context to judge how changes fit existing structure and conventions.
3. **Review against the dimensions below**, reporting only actionable findings.
4. **Produce the review** in the output format below.

## Priority

- **Primary** (the focus of this review): Architecture & Organization, Complexity & Over-engineering, Readability.
- **Blocking regardless**: Correctness, Security — never wave these through.
- **Secondary**: everything else; report when it matters, don't pad the review.

## Primary Dimensions

- **Architecture & Organization** — Is the code in the right layer (Rails: controller vs. service/interactor vs. model; React: component vs. hook vs. util)? Are responsibilities cohesive — no fat controllers, god objects, or business logic leaking into views/components? Does it reuse existing helpers, concerns, and components instead of reinventing them? Are new files/modules justified, or should this live somewhere that already exists? Does it follow the patterns already established in the codebase?

- **Complexity & Over-engineering** — Is this the simplest thing that works? Flag: abstractions with a single caller, premature generalization (params/config/options nothing uses), unnecessary indirection or wrapper layers, design patterns where a plain function or component would do, speculative "future-proofing," and deeply nested logic that could be flattened. Could a new teammate follow it in one pass?

- **Readability** — Clear, consistent names matching codebase conventions; obvious control flow; functions/components that do one thing at a reasonable size; no clever one-liners that obscure intent; comments only where the *why* is non-obvious, never narration of the *what*.

## Secondary Dimensions

- **Correctness** *(blocking)* — logic errors, off-by-one, nil/undefined derefs, bad conditionals, race conditions, queries that don't return what the code expects.
- **Security** *(blocking)* — injection (SQL/XSS), mass assignment, missing/incorrect authz, unvalidated input, exposed secrets or PII.
- **Edge Cases** — empty/nil values, boundaries, concurrent access, sensible defaults.
- **Error Handling** — errors caught at the right level, specific (not overly broad) rescue/catch, meaningful messages, no inconsistent state on failure.
- **Performance** — N+1 queries (preload associations), redundant DB calls, missing indexes, unbounded result sets, unnecessary React re-renders.
- **Test Quality** — happy path AND failures covered, edge cases tested, meaningful assertions, appropriate mocks, minimal setup.
- **Completeness** — acceptance criteria met; migrations, serializers, routes, permissions, feature flags, and docs updated consistently.
- **Backward Compatibility** — API/interface/prop changes don't break consumers; migrations reversible and zero-downtime safe; no dangling references to removed columns.
- **Accessibility (UI only)** — keyboard navigation, screen reader support (ARIA, semantic HTML), contrast and focus indicators.

## Watch For (common over-engineering smells)

- Wrappers/classes that only delegate to one other thing.
- Try/catch or null-guards for states that can't actually occur.
- Re-implemented stdlib, Rails, or React functionality.
- Patterns inconsistent with the rest of the file/module.
- Over-broad params/props/interfaces where only one shape is ever passed.
- Comments that just restate the code.

## Output Format

Be terse. One line per finding. No preamble, no restating the diff, no praise padding. Omit any section with no findings.

```markdown
# Code Review: [branch name]

**Summary:** [1 sentence — overall call + biggest concern.]

### 🔴 Critical
- **[Dimension]** `file:line` — Issue. Fix: [concrete suggestion].

### 🟠 Should Fix
- **[Dimension]** `file:line` — Issue. Fix: [concrete suggestion].

### 🟡 Design — architecture, complexity, readability
- **[Dimension]** `file:line` — Issue. Fix: [concrete suggestion].

### 🟢 Nit
- **[Dimension]** `file:line` — Suggestion.
```

## Guidelines

- Be terse: no intro/outro, no summarizing what the code does beyond the one-line summary.
- Architecture, complexity, and readability findings go in 🟡 Design at minimum (or 🟠/🔴 if severe) — never 🟢 Nit. Reserve 🟢 Nit for trivial style only.
- One line per finding; cite `file:line` and pair each with a concrete fix or question.
- For complexity findings, prefer suggesting what to remove or collapse over what to add.
- Respect project conventions and linter config; skip anything a linter would catch.
- Prioritize design-level findings over nits.
- Ask rather than assume when intent is unclear.
