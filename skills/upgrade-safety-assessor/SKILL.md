---
name: upgrade-safety-assessor
description: Compares release notes (breaking changes, deprecations, migration) to dependency usage and determines upgrade safety (Safe / Needs code changes / High risk). Use when you have a release notes summary and a usage report and need a safety assessment and list of required code changes for the upgrade report.
---

# Upgrade Safety Assessor

Use this skill when you have a **release notes summary** and a **usage report** (from dependency-usage-analyzer) and need to determine upgrade safety and any required code changes. The agent compares release notes to usage and outputs a safety level, which breaking changes affect the codebase, and a "Code changes" list for inclusion in the upgrade report.

## Inputs

- **Release notes summary** (required): Structured list of breaking changes, deprecations, migration/upgrade steps, and security fixes (if any).
- **Usage report** (required): From dependency-usage-analyzer – list of files and how they use the dependency (file:line + description).
- **Current version** and **target version** (optional): Helps interpret version gaps in release notes.

## Workflow

1. **Compare** – For each breaking change and deprecation in the release notes, check whether the usage report shows that this codebase uses the affected API, prop, or pattern.
2. **Assess** – Determine:
   - **Safe** – No breaking changes or deprecations affect this codebase; upgrade can proceed with lockfile/manifest change only.
   - **Needs code changes** – One or more breaking changes or deprecations affect usage; list required code changes with file:line and description.
   - **High risk** – Major API changes, unclear migration, or missing release notes for intermediate versions; note in assessment and open questions.
3. **Output** – Produce:
   - **Safety assessment:** One paragraph: [Safe | Needs code changes | High risk] – reasons tied to release notes and usage.
   - **Affected breaking changes:** Which release-note items affect this codebase (brief list).
   - **Code changes:** For inclusion in the upgrade report. Format: `path/to/file.ext`: description (replace X with Y / add Z / ...). Only include if safety is "Needs code changes" or "High risk" and specific edits are known.

If release notes mention security fixes (e.g. CVE), call that out in the safety assessment.

## Output format

Structure the result for inclusion in the orchestrator's report:

```markdown
## Safety assessment
[Safe | Needs code changes | High risk] – [reasons tied to release notes and usage.]

## Affected breaking changes
- [Brief list of which release-note items affect this codebase]

## Code changes
- `path/to/file.ext`: [description: replace X with Y / add Z / ...]
- ...
```

If no code changes are needed, omit the "Code changes" section or state "None required."

## What NOT to do

- Do not re-analyze the codebase; use only the usage report and release notes summary provided.
- Do not use web search; work only from the inputs passed in.
- Do not perform file edits; only produce the assessment and code-change list for the report.
