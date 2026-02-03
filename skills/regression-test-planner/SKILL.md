---
name: regression-test-planner
description: Produces a regression test plan for a dependency upgrade from affected usage and release notes. Use when you have a usage report and release notes and need existing test commands, areas to verify, manual QA steps, and new test recommendations for the upgrade report.
---

# Regression Test Planner

Use this skill when you have **affected usage** (from dependency-usage-analyzer) and **release notes summary** and need a regression test plan for the upgrade report. The agent outputs existing test commands/paths, areas to verify, manual QA steps, and new test recommendations.

## Inputs

- **Affected usage** (required): List of files and how they use the dependency (file:line + description). From dependency-usage-analyzer.
- **Release notes summary** (required): Breaking changes, deprecations, migration, security. Used to prioritize which flows to test.
- **Ecosystem** (optional): Ruby / npm / yarn / pnpm – helps suggest project-specific test commands (e.g. `yarn test`, `bundle exec rspec`).

## Workflow

1. **Map usage to flows** – From the affected usage list, identify components, features, or flows that use the dependency (e.g. date picker in filters, task deadline, participant onboard).
2. **Identify existing tests** – From project layout and conventions, identify test commands and paths that cover the affected areas (e.g. Jest for components, RSpec for Ruby, Cypress for E2E). Suggest the command(s) to run (e.g. `yarn test`, `yarn test path/to/date_filter_input_spec`).
3. **Areas to verify** – List components, flows, or features that should be manually or automatically verified after the upgrade (e.g. date picker open/close, range selection, snapshot updates).
4. **Manual QA** – If applicable, list manual checks (e.g. "Open date picker in hub filters and select a date").
5. **New tests** – If release notes suggest new behavior or edge cases, recommend adding tests; otherwise state "None recommended" or omit.

## Output format

Structure the result for inclusion in the orchestrator's report:

```markdown
## Regression test plan
- **Existing tests:** [command(s) or paths to run, e.g. yarn test, bundle exec rspec spec/path]
- **Areas to verify:** [components, flows, or features]
- **Manual QA:** [if applicable]
- **New tests:** [if recommended, or "None recommended"]
```

## What NOT to do

- Do not re-analyze the codebase; use only the affected usage and release notes summary provided.
- Do not use web search; work only from the inputs and project conventions (e.g. package.json scripts, test directory layout).
- Do not run tests; only produce the plan for the report.
