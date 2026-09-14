# codelore

**Project documentation as a context layer your AI agent consults on its own: write it, index it, and have the right docs loaded before every plan, fix, or investigation.**

[![Version](https://img.shields.io/badge/version-0.6.0-blue)](.claude-plugin/plugin.json)
[![License](https://img.shields.io/badge/license-Apache--2.0-green)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude_Code-plugin-cc785c)](#install)
[![Codex CLI](https://img.shields.io/badge/Codex_CLI-plugin-1f2328)](#install)

Three skills that turn `docs/` into something an AI session reads at the right moment, without anyone wiring docs into prompts by hand. Part of the [qa-vault](https://github.com/qa-vault/marketplace) plugin family built around [QA Vault](https://qa-vault.com), an MCP-native test-management platform.

## What it does

- **Documents the system, not the work.** Implementation docs describe how a feature works and why it was built that way. No migration history, no "this used to be".
- **Indexes every doc.** Each doc carries YAML frontmatter (`name`, `description`, `triggers`, `related`), and `docs/INDEX.md` is regenerated on every run.
- **Routes docs into context automatically.** When you plan, debug, investigate, or onboard, the router reads the index and pulls in only the docs that match the task.
- **Adopts existing docs in one pass.** A one-shot, idempotent migration adds frontmatter to what you already have and bootstraps the index.

Net effect: docs are written, indexed, and auto-loaded on every relevant task.

## Quick start

```
/plugin marketplace add qa-vault/marketplace
/plugin install codelore@qa-vault
```

In a project that already has docs:

```
/migrate-project-docs
```

After you implement something:

```
/document-feature
```

From then on `consulting-project-docs` loads the relevant docs whenever your prompt is a planning, debugging, or onboarding task. Full steps for both harnesses are under [Install](#install).

## Skills

| Skill | When it runs | What it does |
|---|---|---|
| `document-feature` | "Document what we just implemented", "Write up how the rate limiter works", "Update the docs for the payment module" | Writes or updates an implementation doc with frontmatter and regenerates `docs/INDEX.md`. Documents the system as it is; excludes anything under `docs/plans/`, `plans/`, or `specs/`. |
| `consulting-project-docs` | Before any planning, debugging, investigation, or onboarding task. Usually triggers on its own. | Reads `docs/INDEX.md` and loads only the docs whose `description` or `triggers` match the task. Silent no-op when no index exists. |
| `migrate-project-docs` | "Set up codelore docs in this project", "Migrate the existing docs" | One-shot bulk migration: adds frontmatter to pre-existing docs and bootstraps the index. Idempotent, skips plan folders, asks before touching ambiguous files. |

### Invoking a skill

Explicit mention is recommended. In Claude Code call the slash command; in Codex type `$` in the composer to open the skill-mention popup:

```
/document-feature
$document-feature
```

Both harnesses also auto-detect a skill when your prompt matches its description.

## How it fits with the other qa-vault plugins

- [`quality-loop`](https://github.com/qa-vault/quality-loop) ships `exploratory-qa`, a skeptical reviewer of plans and code. When a `docs/INDEX.md` exists it loads the relevant docs as more material to question, including doc/code drift.
- [`qa-vault-skills`](https://github.com/qa-vault/qa-vault-skills) authors and maintains test cases through the QA Vault MCP. Indexed implementation docs give those skills grounded product context.

Neither plugin requires codelore, and codelore requires neither of them.

## Install

`codelore` is distributed through the `qa-vault` marketplace catalog. Installing it is self-contained: no other `qa-vault` plugin is required.

<details>
<summary><strong>Claude Code</strong></summary>

1. **Add the marketplace** (one-time):

   ```
   /plugin marketplace add qa-vault/marketplace
   ```

   This fetches the catalog of `qa-vault` plugins from GitHub. No code is installed yet. If you already added it for another `qa-vault` plugin, skip this step.

2. **Install the plugin**:

   ```
   /plugin install codelore@qa-vault
   ```

   Claude Code asks where to install:
   - **User**: available in every project on your machine (recommended for personal use)
   - **Project**: only active in this project, shared with teammates via `.claude/settings.json`
   - **Local**: only for you, only in this project

3. **Verify**: type `/` and you should see `/document-feature`, `/consulting-project-docs`, and `/migrate-project-docs`, each annotated `(codelore)`.

**Updates:** Claude Code auto-updates installed plugins at startup.

</details>

<details>
<summary><strong>Codex CLI</strong></summary>

> Requires Codex CLI 0.122 or later. The `url` source variant this catalog uses shipped in stable 0.122 (2026-04-20).

1. **Add the marketplace** (one-time):

   ```
   codex plugin marketplace add qa-vault/marketplace
   ```

   If you already added it for another `qa-vault` plugin, skip this step.

2. **Install the plugin**: inside Codex, open the plugin browser:

   ```
   /plugins
   ```

   Find `codelore` under the `qa-vault` marketplace and toggle it on. `/plugins` is an interactive browser and does not accept inline arguments.

3. **Verify**: type `$` in the Codex composer to open the skill-mention popup. `document-feature`, `consulting-project-docs`, and `migrate-project-docs` should be listed.

**Updates:** refresh with `codex plugin marketplace upgrade qa-vault` periodically.

</details>

## License

Apache-2.0. See [LICENSE](LICENSE).
