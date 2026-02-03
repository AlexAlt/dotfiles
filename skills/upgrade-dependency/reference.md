# Upgrade dependency – reference

Use this file for project-specific or detailed upgrade commands. The agent may read it when ecosystem or tooling details are needed.

## Ruby / Bundler

- **Lockfile:** `Gemfile.lock` (version under `GEM` and under the gem name).
- **Update one gem:** `bundle update <gem>` (respects version in `Gemfile`).
- **Set version in Gemfile:** `gem "<name>", "~> X.Y.Z"` then `bundle update <name>`.
- **Multiple Gemfiles:** Check for `Gemfile`, `*/Gemfile`, or app-specific Gemfiles in monorepos.

## npm / yarn / pnpm

- **Lockfile:** `yarn.lock`, `package-lock.json`, or `pnpm-lock.yaml`.
- **Update one package:** `yarn add <pkg>@<version>`, `npm install <pkg>@<version>`, or `pnpm add <pkg>@<version>`.
- **Workspaces:** Check root and workspace `package.json`; run install from root.
- **Peer dependencies:** If release notes require a different peer (e.g. React), plan a separate upgrade or note it in the upgrade steps.

## Rollback

- **Ruby:** Revert `Gemfile` and `Gemfile.lock`, then `bundle install`.
- **npm/yarn/pnpm:** Revert `package.json` and lockfile, then `yarn install` / `npm install` / `pnpm install`.
