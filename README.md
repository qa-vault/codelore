# codelore

Four skills that turn project documentation into a context layer AI sessions consult automatically. The loop:

- **`document-feature`** — writes and maintains implementation docs with YAML frontmatter (`name`, `description`, `triggers`, `related`) and regenerates `docs/INDEX.md` on every run.
- **`migrate-project-docs`** — one-shot bulk migration that adds frontmatter to pre-existing docs and bootstraps `docs/INDEX.md`. Idempotent. Skips anything under `docs/plans/`, `plans/`, or `specs/`.
- **`consulting-project-docs`** — router that reads `docs/INDEX.md` and pulls only the relevant docs into the agent's context whenever you plan, debug, investigate, or onboard. Silent no-op if no index exists.
- **`exploratory-qa`** — skeptically reviews **existing code OR an implementation plan**, surfacing non-obvious decisions, unusual implementations, and architectural choices worth discussing. Treats loaded docs as more material to question (including surfacing doc/code drift), never as authority. Works in two modes:
  - **Code mode** — review an already-implemented feature, module, or file.
  - **Plan mode** — pressure-test an implementation plan, spec, design doc, or RFC **before** code is written. Plans are the cheapest place to catch issues.

Net effect: docs are written → indexed → auto-loaded → critically re-examined on every relevant task — without anyone manually wiring docs into prompts.

This plugin installs natively in both **Claude Code** and **Codex CLI**.

## Contents

- [Install](#install)
  - [Claude Code](#claude-code)
  - [Codex CLI](#codex-cli)
- [Using the skills](#using-the-skills)
- [License](#license)

---

<img src="https://img.shields.io/badge/Install-2ea043?style=for-the-badge" alt="Install" />

## Install

Pick the section matching your AI coding tool.

<img src="https://img.shields.io/badge/Claude_Code-cc785c?style=for-the-badge" alt="Claude Code" />

### Claude Code

Claude Code has a built-in plugin system. You add the `qa-vault` marketplace once, then install `codelore` from it.

1. **Add the marketplace** (one-time):

   ```
   /plugin marketplace add qa-vault/marketplace
   ```

   This fetches a catalog of `qa-vault` plugins from GitHub. No code is installed yet.

2. **Install the plugin**:

   ```
   /plugin install codelore@qa-vault
   ```

   Claude Code will ask where to install:
   - **User** — available in every project on your machine (recommended for personal use)
   - **Project** — only active when you open this project, and shared with teammates via `.claude/settings.json`
   - **Local** — only for you, only in this project

3. **Verify** — type `/` and you should see `/exploratory-qa`, `/document-feature`, `/consulting-project-docs`, and `/migrate-project-docs` in the list (each annotated `(codelore)` so you can tell where they come from).

**Updates:** Claude Code auto-updates installed plugins at startup. Nothing to do on your side.

<img src="https://img.shields.io/badge/Codex_CLI-1f2328?style=for-the-badge" alt="Codex CLI" />

### Codex CLI

Codex also has a plugin marketplace system (since March 2026). The install flow mirrors Claude Code's.

> **Requires Codex CLI 0.122+.** The `url` source variant used here shipped in stable 0.122 (2026-04-20). Earlier 0.121.x releases accept only `local` plugin sources and cannot install polyrepo catalogs like this one — upgrade to 0.122 or later.

1. **Add the marketplace** (one-time):

   ```
   codex plugin marketplace add qa-vault/marketplace
   ```

2. **Install the plugin**:

   Inside Codex, open the plugin browser:

   ```
   /plugins
   ```

   Find `codelore` under the `qa-vault` marketplace and toggle it on to install. (`/plugins` is an interactive browser — it does not accept inline arguments.)

3. **Verify** — type `$` in the Codex composer to open the skill-mention popup; `exploratory-qa`, `document-feature`, `consulting-project-docs`, and `migrate-project-docs` should all be listed. Invoke a skill explicitly with `$<skill-name> <your request>`. As a fallback, Codex will auto-detect a skill when your prompt matches its `description` (see "Using the skills" below for example phrases).

**Updates:** refresh with `codex plugin marketplace upgrade qa-vault` periodically. (Codex's auto-update behavior on launch is not documented as of April 2026, so manual refresh is the reliable path.)

---

<img src="https://img.shields.io/badge/Using_the_skills-0969da?style=for-the-badge" alt="Using the skills" />

## Using the skills

Two ways to invoke each skill — explicit is recommended.

**1. Explicit mention (recommended).**

- In **Codex**, type `$` in the composer to open the skill-mention popup, then select the skill and add your request:

  ```
  $exploratory-qa the notification service
  $document-feature
  ```

- In **Claude Code**, call them by slash command:

  ```
  /exploratory-qa the notification service
  /document-feature
  ```

**2. Auto-detect (fallback).** If you don't mention the skill explicitly, both Claude Code and Codex will pick one when your prompt matches the skill's `description`. Useful phrases:

| Skill | Mode | Example prompts |
|---|---|---|
| `exploratory-qa` | Code | "Explore the checkout flow", "QA this module critically", "Review src/auth/ with a skeptical eye", "What's non-obvious here?" |
| `exploratory-qa` | Plan | "QA this plan: plans/rate-limiting.md", "Critique this spec before I implement it", "Review this design doc with a skeptical eye", "What's missing from this RFC?" |
| `document-feature` | — | "Document what we just implemented", "Write up how the rate limiter works", "Update the docs for the payment module" |
| `consulting-project-docs` | — | "How does the auth module work?", "Plan a change to the payment flow", "Investigate why the queue is dropping events", "Onboard me onto this codebase" — usually triggers on its own before any planning/debugging task. |
| `migrate-project-docs` | — | "Set up codelore docs in this project", "Migrate the existing docs", "Add frontmatter to my docs and build the index" — also nudged by `consulting-project-docs` on first run when `docs/INDEX.md` is missing. |

---

<img src="https://img.shields.io/badge/License-6e7781?style=for-the-badge" alt="License" />

## License

MIT — see [LICENSE](./LICENSE).
