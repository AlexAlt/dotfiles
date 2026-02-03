---
name: upgrade-dependency
description: Orchestrates upgrading a library dependency (Ruby gem or npm package) by ingesting release notes, spawning sub-skills to analyze usage and assess safety, and producing an execution-ready markdown report. Use when the user wants to upgrade a dependency, evaluate an upgrade's impact, or plan regression testing. When the user asks to perform the upgrade, invokes execute-dependency-upgrade with the report.
---

# Upgrade Dependency (Orchestrator)

Use this skill when upgrading a library dependency (Ruby gem or npm/yarn package). The agent gathers inputs, fetches release notes, spawns sub-skills to analyze usage and assess safety, and synthesizes an execution-ready markdown report. When the user asks to **perform the upgrade** or **do the upgrade**, invoke **execute-dependency-upgrade** with the path to that report.

## Required inputs

Gather from the user (or infer where possible):

| Input | Required | Example |
|-------|----------|---------|
| **Dependency name** | Yes | `react-datepicker`, `rspec`, `rails` |
| **Target version** | Yes | `4.25.0`, `7.1.0` |
| **Release notes URL(s)** | Yes (or discover) | GitHub releases, changelog URL |

**Release notes:** Accept one link initially. If the user will provide more links (e.g. per major version), say so and process the first, then ask for the next. Support paging: if the page has "Older releases" / "Next" / a version list, fetch or follow those links until coverage from current version to target is sufficient. If versions are missing, note "Release notes missing for vX–vY" in the safety assessment.

## Workflow

Copy this checklist and track progress:

```
- [ ] Step 1: Gather inputs (dependency name, target version, release notes URL)
- [ ] Step 2: Ingest release notes (no web search if URL inaccessible)
- [ ] Step 3: Detect ecosystem and current version from lockfile/manifest
- [ ] Step 3b: If patch-only upgrade (same major.minor, only patch changed), skip to Step 5 (light report); else continue to Step 4
- [ ] Step 4: Spawn subagents (dependency-usage-analyzer, upgrade-safety-assessor, regression-test-planner)
- [ ] Step 5: Synthesize markdown report and save to file
- [ ] Step 6: If user asks to perform upgrade, spawn execute-dependency-upgrade with report path
```

**Patch-only shortcut:** If the target version differs from the current version only in the patch segment (same major and minor; e.g. 7.6.0 → 7.6.1), skip Step 4 entirely. Do not run dependency-usage-analyzer, upgrade-safety-assessor, or regression-test-planner. Proceed from release notes (Step 2) and current/target (Step 3) directly to Step 5, producing a lighter report that summarizes release notes and lists upgrade steps only. This avoids wasting resources on trivial patch upgrades.

### Step 1: Gather inputs

Confirm dependency name, target version, and at least one release notes URL. If the user will provide release notes iteratively, accept the first link and ask for "next page" or "next version" as needed.

### Step 2: Ingest release notes

- Fetch each URL (e.g. `mcp_web_fetch` or browser tools).
- **If the URL cannot be accessed** (fetch fails, timeout, 404, or content is unavailable): **Do NOT use web search.** Ask the user to either (1) provide the release notes content by pasting it, or (2) provide an alternate URL. Proceed only when release notes content is available.
- For multiple URLs or paged content: process each page; merge breaking/migration sections.
- Extract and structure:
  - **Breaking changes**
  - **Deprecations**
  - **Migration / upgrade guides**
  - **Security fixes** (CVEs) if present

If versions are missing between current and target, state the gap and note uncertainty in the safety assessment.

### Step 3: Detect ecosystem and current version

- **Ruby:** Read `Gemfile`, `Gemfile.lock`, or `.gemspec`; current version from lockfile.
- **npm/yarn:** Read `package.json`, `yarn.lock` or `package-lock.json`; current version from lockfile.
- Handle monorepos (multiple lockfiles or workspaces). If current version is unknown, say so and continue.

### Step 3b: Patch-only check

Compare current and target versions (semver: major.minor.patch). If **major and minor are identical** and only the patch segment changes (e.g. 7.6.0 → 7.6.1), treat this as a **patch-only upgrade**.

- **Patch-only:** Skip Step 4. Do not invoke dependency-usage-analyzer, upgrade-safety-assessor, or regression-test-planner. Go to Step 5 and produce a **light report** (release notes summary, upgrade steps, optional brief regression note). Use the same output template but for **Affected usage** write "Patch-only upgrade; codebase analysis skipped." and for **Safety assessment** use "Patch-only – low risk; see release notes summary."
- **Not patch-only** (minor or major bump, or current unknown): Continue to Step 4 and run the full workflow.

### Step 4: Spawn subagent tasks

Create subagent tasks in parallel. Pass the dependency name, target version, and (where needed) the release notes summary and usage report.

**Task 1 – dependency-usage-analyzer**

Use **dependency-usage-analyzer** to find and document all usage of [dependency name] in this codebase. Pass: dependency name, optional ecosystem.

**Task 2 – upgrade-safety-assessor**

Use **upgrade-safety-assessor** with the release notes summary (breaking changes, deprecations, migration, security) and the usage report from dependency-usage-analyzer to determine safety and required code changes. Pass: release notes summary, usage report (from Task 1). Wait for Task 1 output before starting, or pass both when Task 1 completes.

**Task 3 – regression-test-planner**

Use **regression-test-planner** with the affected usage list and release notes to produce a regression test plan. Pass: affected usage (from Task 1), release notes summary, ecosystem.

Wait for **all** subagent tasks to complete before proceeding.

### Step 5: Synthesize report and save

- **Full path (Step 4 was run):** Combine the outputs from dependency-usage-analyzer, upgrade-safety-assessor, and regression-test-planner into one markdown report using the **Output format** below.
- **Patch-only path (Step 4 skipped):** Build the report from release notes (Step 2) and current/target (Step 3) only. Use the same **Output format** but set **Affected usage** to "Patch-only upgrade; codebase analysis skipped.", **Safety assessment** to "Patch-only – low risk; see release notes summary.", and **Regression test plan** to the same table structure with one or two rows (e.g. run existing test suite; optional manual smoke check).
- Save the report to a markdown file (e.g. `UPGRADE-<dependency-name>-<version>.md` in the upgrade-dependency skill directory, or a path the user specifies).
- This file can be pasted into or read by the **execute-dependency-upgrade** skill to perform the upgrade when the user asks.

### Step 6: Perform upgrade (when user asks)

If the user asks to **perform the upgrade** or **do the upgrade**, use **execute-dependency-upgrade** with the path to the generated report: [path to the saved markdown file].

## Output format

Structure the final report so it is both human-readable and parseable by **execute-dependency-upgrade**. Use this template:

```markdown
# Upgrade: [dependency name] → [target version]

## Summary
[1–2 sentence safety summary and main actions.]

## Current vs target
- Current: [version or "unknown"]
- Target: [version]
- Ecosystem: [Ruby / npm / yarn / pnpm]

## Release notes summary
- Breaking changes: [brief list]
- Deprecations: [brief list]
- Migration / upgrade: [key steps]
- Security: [if any]

## Affected usage
- `path/to/file.ext:line` – [how it uses the dependency]
- ...

## Safety assessment
[Safe | Needs code changes | High risk] – [reasons.]

## Upgrade steps
1. Edit [package.json]: change "[dependency]": "[old]" to "[dependency]": "[new]".
2. Run: [yarn install | bundle update <gem> | ...]
3. [If code changes needed] Apply code changes below.

## Code changes
- `path/to/file.ext`: [description: replace X with Y / add Z / ...]
- ...

## Regression test plan

**Existing tests:** [command(s) or paths, e.g. yarn test, bundle exec rspec spec/path]

| Area / location | What to verify | Notes |
|-----------------|----------------|-------|
| [component, flow, or feature] | [specific checks: open/close, selection, layout, etc.] | Manual QA / Covered by tests / New test recommended |
| ... | ... | ... |

**New tests:** [if recommended, or "None recommended"]

## Open questions / gaps
[Missing versions, unclear migration, etc.]
```

**Upgrade steps** must be a numbered list. Each step is either:
- **Manifest edit:** `Edit <file path>: change <old> to <new>` (execute-dependency-upgrade parses and applies).
- **Command:** `Run: <command>` (e.g. `Run: yarn install`).
- **Code change:** Reference to "Code changes" section (e.g. "Apply code changes below.").

**Code changes** lists each change so execute-dependency-upgrade can apply it: `path/to/file.ext`: description or "replace X with Y".

## Ecosystem reference

- **Ruby:** Version in `Gemfile` or `Gemfile.lock`. Update: change `Gemfile` if needed, then `bundle update <gem>`.
- **npm/yarn:** Version in `package.json` and lockfile. Update: `npm install <pkg>@<version>` or `yarn add <pkg>@<version>` (or equivalent for pnpm).

For detailed or project-specific commands, see [reference.md](reference.md) if present.

## What NOT to do

- Do not hardcode package names or URLs in the skill.
- Do not assume a single lockfile (check for monorepos/workspaces).
- Do not use web search as a fallback when the release notes URL is inaccessible; ask the user for the content or an alternate link.
- Do not run destructive or write commands in this skill unless the user has asked to perform the upgrade (then invoke execute-dependency-upgrade instead).
- Do not skip cross-referencing release notes with actual usage; safety must be based on both.
