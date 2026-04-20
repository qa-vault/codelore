# codelore

Two skills that help you **build and preserve understanding of a codebase**:

- **`exploratory-qa`** — a skeptical reviewer that reads a feature and surfaces non-obvious decisions, unusual implementations, and architectural choices worth discussing. Not a linter or bug finder — a thinking agent that asks _"why?"_
- **`document-feature`** — creates and maintains living reference docs that explain how a feature works, why it was built that way, and what to watch out for when modifying it.

Use them when onboarding to unfamiliar code, critically reviewing a module, or wrapping up a feature that deserves documentation.

This plugin installs natively in both **Claude Code** and **Codex CLI**.

---

## Install

Pick the section matching your AI coding tool.

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

3. **Verify** — type `/` and you should see `/codelore:exploratory-qa` and `/codelore:document-feature` in the list.

**Updates:** Claude Code auto-updates installed plugins at startup. Nothing to do on your side.

### Codex CLI

Codex also has a plugin marketplace system (since March 2026). The install flow mirrors Claude Code's.

1. **Add the marketplace** (one-time):

   ```
   codex marketplace add qa-vault/marketplace
   ```

2. **Install the plugin**:

   ```
   /plugins install codelore
   ```

   (Run inside Codex. You can also browse via the `/plugins` command.)

3. **Verify** — Codex will auto-detect the skills. You can trigger them with natural phrases (see "Using the skills" below).

**Updates:** refresh with `codex marketplace update qa-vault` periodically. (Codex's auto-update behavior on launch is not documented as of April 2026, so manual refresh is the reliable path.)

---

## Using the skills

Once installed, you don't need to invoke them explicitly — both skills trigger automatically on natural phrases:

| Skill | Example prompts |
|---|---|
| `exploratory-qa` | "Explore the checkout flow", "QA this module critically", "Review src/auth/ with a skeptical eye", "What's non-obvious here?" |
| `document-feature` | "Document what we just implemented", "Write up how the rate limiter works", "Update the docs for the payment module" |

In Claude Code you can also call them by explicit name:

```
/codelore:exploratory-qa the notification service
/codelore:document-feature
```

---

## License

MIT — see [LICENSE](./LICENSE).
