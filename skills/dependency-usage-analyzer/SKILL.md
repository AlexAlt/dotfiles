---
name: dependency-usage-analyzer
description: Finds and documents all usage of a library dependency in the codebase using codebase-locator and codebase-analyzer. Use when you need a structured list of files and how they use a dependency (APIs, props, config). Output is for upgrade-safety-assessor and regression-test-planner. No safety judgment; documentation only.
---

# Dependency Usage Analyzer

Use this skill when you need to find and document **how** a dependency is used across the codebase. The agent uses codebase-locator to find all references, then codebase-analyzer to document usage. Output is a structured list of affected usage (file:line + description) for use by upgrade-safety-assessor and regression-test-planner.

## Inputs

- **Dependency name** (required): e.g. `react-datepicker`, `rspec`, `rails`
- **Ecosystem** (optional): Ruby / npm / yarn / pnpm (helps narrow lockfile and manifest paths)

## Workflow

1. **Locate** – Use **codebase-locator** (or equivalent): grep/glob for the dependency name, `require`/`import` statements, and lockfile/manifest entries. Produce a list of files and entry points (components, Gemfile, config, CSS targeting the dependency's classes).
2. **Analyze** – Use **codebase-analyzer** (or **codebase-research** for many or cross-cutting files): For each located file (or a representative set), document HOW the dependency is used – APIs, props, config, patterns. Include file:line references.
3. **Output** – Return a structured list in this form:

```markdown
## Affected usage
- `path/to/file.ext:line` – [how it uses the dependency: APIs, props, config]
- ...
```

Do not evaluate safety or suggest code changes; only document what exists.

## Output format

Structure the result as:

- **Affected usage:** A list of entries: `` `path/to/file.ext:line` – description of how the dependency is used (APIs, props, config). ``

Include lockfile/manifest paths (e.g. `package.json`, `Gemfile.lock`) and any CSS or tests that reference the dependency's public API or class names.

## What NOT to do

- Do not assess whether the upgrade is safe or recommend code changes.
- Do not use web search; work only from the codebase and any context passed in (e.g. dependency name, ecosystem).
- Do not skip test files, config, or CSS that targets the dependency.
