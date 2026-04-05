---
name: lefthook
description: Create, explain, review, and troubleshoot Lefthook hook configurations for Git repositories. Use when Codex needs to author or update `lefthook.yml`, `.lefthook.yml`, `.config/lefthook.yml`, `lefthook-local.*`, explain `commands` vs `jobs` vs `scripts`, wire up `install`, `run`, `dump`, or `validate`, add hook automation such as linting, formatting, tests, commit message checks, or local overrides, or debug why a Lefthook setup does not behave as expected.
---

# Lefthook

Create or update Lefthook configuration with exact option names, practical defaults, and clear debugging steps. Explain the tool briefly, then produce a working config for the current repository.

Use this skill when the task mentions:

- `lefthook.yml`, `.lefthook.yml`, `.config/lefthook.yml`, `lefthook-local.yml`, or another `lefthook-local.*` file
- Git hooks such as `pre-commit`, `commit-msg`, `pre-push`, `post-merge`, or `post-checkout`
- `lefthook install`, `lefthook run`, `lefthook dump`, or `lefthook validate`
- choosing between `commands`, `jobs`, and `scripts`
- placeholders such as `{staged_files}`, `{push_files}`, `{all_files}`, `{files}`, or `{1}`
- hook bugs such as commands not running, wrong files matching, fixed files not restaged, or local-only wrappers leaking into shared config

## Non-Negotiables

1. Inspect the repository before writing config. Reuse the existing config filename and format if one already exists.
2. Match hook commands to the repo's real toolchain. Reuse existing package manager commands, scripts, and linters instead of inventing wrappers.
3. Choose the simplest structure that fits. Use `commands` by default, `jobs` for ordering or grouping, and `scripts` only for executable hook files.
4. Use placeholders and filters that match the hook type. `pre-commit` usually wants `{staged_files}`, `pre-push` usually wants `{push_files}`, and `commit-msg` usually wants `{1}`.
5. Add `stage_fixed: true` only for commands that rewrite tracked files and should re-stage them.
6. Put local-only skips, Docker wrappers, and private overrides in `lefthook-local.*`, not in the shared team config.
7. When debugging or reviewing, use `lefthook validate`, `lefthook dump`, and `lefthook run <hook>` before guessing.

## Start Here

Inspect the repository before writing anything.

- Look for existing config files first: `lefthook.yml`, `.lefthook.yml`, `.config/lefthook.yml`, `lefthook.yaml`, `lefthook.toml`, `lefthook.json`, `lefthook.jsonc`, and matching `lefthook-local.*` overrides.
- Preserve the existing format and naming if a config already exists.
- Prefer YAML `lefthook.yml` for new setups unless the repo already standardizes on TOML, JSON, or JSONC.
- Read package manager files, script directories, and CI or lint tooling before proposing hook commands.
- Keep one main format per project. Do not introduce a second main Lefthook config format.

## Author Configs

Choose the simplest structure that fits the workflow.

- Use `commands` for the default case: straightforward named tasks such as linting, formatting, tests, or commit message checks.
- Use `jobs` when order, grouping, `parallel`, `piped`, mixed `run` and `script` entries, or merge-friendly named jobs matter.
- Use `scripts` only when the repository already uses executable files under `source_dir/<hook>/` or when the user explicitly wants file-based scripts.
- Prefer hook names that match real Git hooks such as `pre-commit`, `commit-msg`, and `pre-push`. It is also valid to define custom task groups and run them through `lefthook run <name>`.
- Keep commands local to the repo toolchain. Reuse existing `npm`, `pnpm`, `yarn`, `bun`, `go`, `bundle`, `uv`, or shell commands instead of inventing new wrappers.
- Use file templates deliberately: `{staged_files}` for `pre-commit`, `{push_files}` for `pre-push`, `{all_files}` for whole-repo checks, `{files}` when a custom `files:` command is defined, `{cmd}` for local wrapping, and `{1}` for hook arguments such as `commit-msg`.
- Add `stage_fixed: true` only when the command actually rewrites tracked files and should re-stage them.
- Use `glob`, `exclude`, `file_types`, and `root` to keep commands narrow and fast.
- Reach for `lefthook-local.*` when the user wants local-only skips, Docker wrappers, or private overrides that should not change the shared team config.

Integrated example for a JS or TS repo that already uses ESLint, Prettier, and Commitlint:

```yml
pre-commit:
  commands:
    eslint:
      glob: "*.{js,jsx,ts,tsx}"
      run: pnpm eslint --fix {staged_files}
      stage_fixed: true
    prettier:
      glob: "*.{js,jsx,ts,tsx,json,md,yml,yaml}"
      run: pnpm prettier --write {staged_files}
      stage_fixed: true

commit-msg:
  commands:
    commitlint:
      run: pnpm commitlint --edit {1}
```

Why this shape:

- `commands` is enough because each task is independent.
- `pre-commit` uses `{staged_files}` because the checks should scope to the staged set.
- `commit-msg` uses `{1}` because Git passes the commit message file path as the first hook argument.
- Both formatters set `stage_fixed: true` because they rewrite tracked files.

Read [overview.md](./references/overview.md) for install and command flow, [config-model.md](./references/config-model.md) for structure and merge behavior, [patterns.md](./references/patterns.md) for common recipes, [best-practices.md](./references/best-practices.md) when the user asks for battle-tested guidance, and [schema-cheatsheet.md](./references/schema-cheatsheet.md) for exact keys.

## Explain And Debug

Explain Lefthook in terms of repository behavior, not marketing.

- Describe it as a Git hook manager that reads config on each run and executes repo-defined commands or scripts.
- Mention that `lefthook install` installs Git hook entrypoints and creates an empty config if none exists.
- Mention that config changes do not require reinstall in normal cases because hooks read config each time they run.
- Use `lefthook validate` to check configuration validity.
- Use `lefthook dump` to inspect the effective merged config after `extends`, `remotes`, and `lefthook-local.*`.
- Use `lefthook run <hook>` with `--all-files`, `--file`, `--job`, or `--tag` to reproduce hook behavior outside `git commit` or `git push`.
- Explain merge precedence as: main config, then `extends`, then `remotes`, then `lefthook-local.*`.
- When using the phrase "best practice", ground exact behavior in the bundled docs and schema, and treat blog-derived patterns as workflow guidance rather than hard requirements.

When reviewing an existing config, call out:

- commands that will not match any files because placeholders and filters do not align;
- rewriters that forgot `stage_fixed: true`;
- file-wide commands incorrectly using `{staged_files}`;
- unnecessary complexity where `commands` would be enough;
- local overrides that should live in `lefthook-local.*` instead of shared config.

## Use Bundled Resources

- For a quick starting point, copy `assets/minimal-pre-commit.yml`. For JS or TS repos, copy `assets/js-ts-pre-commit.yml`. Then adapt the package manager and commands to the repo.
- Load `references/patterns.md` for concrete recipes before drafting a non-trivial config.
- Load `references/best-practices.md` when the user explicitly asks for recommendations or wants rationale backed by real-world Lefthook usage.
- Load `references/schema-cheatsheet.md` when exact option spelling or allowed nesting matters.
- Keep explanations compact in the final answer. Include only the options that matter for the current repository.
