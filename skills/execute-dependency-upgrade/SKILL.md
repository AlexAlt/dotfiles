---
name: execute-dependency-upgrade
description: Performs a dependency upgrade by reading an upgrade report markdown file and executing its Upgrade steps and Code changes sections. Use when the user has run upgrade-dependency and asks to perform the upgrade or do the upgrade. Does not re-analyze or re-fetch release notes; only executes what the report specifies.
---

# Execute Dependency Upgrade

Use this skill when the user has an **upgrade report** (from upgrade-dependency) and asks to **perform the upgrade** or **do the upgrade**. The agent reads the report, parses "Upgrade steps" and "Code changes", and performs manifest edits, lockfile command, and any code edits. It does **not** re-analyze or re-fetch release notes; it only executes what the report specifies.

## Inputs

- **Path to the upgrade report markdown file** (required): e.g. `.cursor/skills/upgrade-dependency/UPGRADE-react-datepicker-9.1.0.md`, or a path the user provides.
- Alternatively: the **markdown content** of the report (pasted or in context) if the file path is not available.

## Workflow

1. **Read the report** – Read the full markdown file (or use the provided content). Locate the sections **## Upgrade steps** and **## Code changes**.
2. **Parse Upgrade steps** – For each numbered step:
   - **Manifest edit:** If the step says "Edit [file]: change X to Y", open the file and apply the change (e.g. in `package.json` or `Gemfile`).
   - **Command:** If the step says "Run: [command]", run the command (e.g. `yarn install`, `bundle update <gem>`).
   - **Code change:** If the step references "Apply code changes below" or "Code changes", proceed to the Code changes section.
3. **Parse Code changes** – For each entry under **## Code changes**:
   - Format: `path/to/file.ext`: description (replace X with Y / add Z / ...).
   - Read the file, apply the described change (search_replace or edit), and save.
4. **Regression test** – If the report's **Regression test plan** includes an "Existing tests" command, run it (e.g. `yarn test`). If tests fail (e.g. snapshot diffs), report and optionally suggest updating snapshots (e.g. `yarn test -u [path]`).
5. **Output** – Summarize what was done: files edited, commands run. Remind the user of any remaining regression steps from the report (e.g. manual QA, areas to verify).

## Parsing rules

- **Upgrade steps:** Look for lines like:
  - `Edit package.json: change "react-datepicker": "^7.6.0" to "react-datepicker": "^9.1.0".`
  - `Run: yarn install`
  - `Apply code changes below.`
- **Code changes:** Look for lines like:
  - `` `path/to/file.ext`: replace X with Y ``
  - `` `path/to/file.ext`: add Z at line N ``
  Use the description to perform the edit (read file, then search_replace or equivalent).

## Output format

After execution, report:

```markdown
## Summary of changes
- **Manifest:** [file and change made]
- **Lockfile:** [command run]
- **Code changes:** [list of files and changes]
- **Tests run:** [command and result, e.g. passed / snapshot update suggested]

## Next steps (from report)
- [Reminder of manual QA, areas to verify, or new tests from the report's Regression test plan]
```

## What NOT to do

- Do not re-analyze the codebase or re-fetch release notes; only execute what the report specifies.
- Do not add or remove upgrade steps; follow the report.
- Do not skip the Code changes section if the report includes it; apply each listed change.
