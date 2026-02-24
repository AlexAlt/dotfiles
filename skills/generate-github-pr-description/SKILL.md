---
name: generate-github-pr-description
description: Generate GitHub PR description from git changes (traditional git workflow). Use when the user wants a PR description, uses /create_pr_description, or is preparing to open a pull request. Analyzes committed changes since main and fills the project PR template.
---

# Generate GitHub PR Description

Generate a GitHub PR description by analyzing git changes and filling the project's PR template. Uses only git read commands (no push, commit, or checkout).

## Context to Gather

Run these in the repo root (read-only):

- **Current branch:** `git branch --show-current`
- **Branch point:** `git merge-base HEAD origin/main`
- **Commits since main:** `git log origin/main..HEAD --oneline`
- **Commit subjects:** `git log origin/main..HEAD --format="%s"`
- **File change summary:** `git diff origin/main..HEAD --stat`
- **Changed files:** `git diff origin/main..HEAD --name-only`

## Analysis Guidelines

1. **Read shared rules:** [.claude/shared/analysis-guidelines.md](.claude/shared/analysis-guidelines.md) for Change Classification and Smart Description Generation.
2. **Commit style:**
   - Many small commits → focus on overall change theme.
   - Few large commits → use individual commit messages for detail.

## Voice
When generate the PR description command, PR descriptions should use the user voice- casual and trying to keep things light and fun. User tends to use too many exclamation points and self-deprecation.
Example PR description:

Description

In order to disable the inputs when editing a current price, we need to pass the "used" property to the FE. The designs also include the starts_at, so I did add that here, although I think we could probably get away without it, if we think it's extraneous.
Starting with the BE stuff first so we can sort out what we want to include in the response

---


Test Plan
Just specs!!!!!


Description

I kept running into behavior where the TZ offset was changing the dates to be off by 1 day, so I removed all references to time in the dates. This should still be handled appropriately in the FE, and the backend will "normalize" the starts_at date before persisting it to the DB so it at the beginning of day. I think this makes sense and makes me happy!

---


Test Plan

Verify that the dates from the DB appear correctly in the API response and in the DOM

## PR Template

Use the structure from [docs/pull_request_template.md](docs/pull_request_template.md):

- **Description** – Synopsis of the change (and "Closes: #issue" if applicable).
- **Test Plan** – Step-by-step verification.
- **Checklist/Notes** – Optional notes for reviewers (template checklist can be included).

## Implementation Steps

1. Run the git context commands above.
2. Classify changes by type (backend, frontend, database, config, docs, testing) per analysis guidelines.
3. Extract purpose and main theme from commit messages.
4. Propose 1–2 PR title options.
5. Write Description, Test Plan, and Checklist/Notes using the template.
6. Output in the required format below.

## Output Format

**CRITICAL:** Preserve markdown so the user can copy it into GitHub.

1. **Analysis** – Short summary of what changed (scope, file types, main theme).
2. **Suggested PR Title** – 1–2 title options.
3. **Ready-to-copy markdown** – A fenced code block containing the full PR description in markdown, matching [docs/pull_request_template.md](docs/pull_request_template.md).

Example structure for the code block:

```markdown
### Description

<!-- Synopsis -->

---

Closes:

### Test Plan

<!-- Steps -->

### Checklist/Notes

<!-- Optional -->
- [ ] ...
```

Focus on accuracy and usefulness for traditional git branch workflows.
