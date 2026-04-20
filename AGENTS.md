# Agent Context: codelore

This repository is a **dual-ecosystem plugin** — installable natively in both Claude Code and Codex CLI. When working on this codebase, keep both ecosystems in sync.

## Repository layout

```
codelore/
├── .claude-plugin/plugin.json     # Claude Code manifest
├── .codex-plugin/plugin.json      # Codex manifest
├── skills/
│   ├── exploratory-qa/SKILL.md
│   └── document-feature/
│       ├── SKILL.md
│       └── references/doc_template.md
├── README.md
├── LICENSE
└── AGENTS.md                       # this file
```

The `skills/` directory is the **single source of truth** for skill content. Both manifests reference it — never duplicate skill files.

## Cross-platform invariants

These rules must hold at all times. Breaking any of them ships a broken release.

1. **`version` field must match** in `.claude-plugin/plugin.json` and `.codex-plugin/plugin.json`. Both manifests are bumped together.
2. **`description` should match** across both manifests (copy the authoritative one).
3. **`SKILL.md` frontmatter** must include `name` and `description` at minimum — these are the only fields Codex requires, and Claude Code accepts extras without complaint.
4. **Never add a skill to just one ecosystem.** All skills live under `skills/<name>/` and are auto-discovered by both tools.
5. **Don't hard-code tool-specific paths** inside `SKILL.md`. Skill content must be written in a way that works in either runtime.

## Adding a new skill

1. Create `skills/<kebab-case-name>/SKILL.md` with YAML frontmatter (`name`, `description`) and body content.
2. If the skill needs supporting files, add them in the same directory (e.g. `skills/<name>/references/`).
3. Bump `version` in BOTH `.claude-plugin/plugin.json` and `.codex-plugin/plugin.json` (minor bump for additive changes, per semver).
4. Update the README skill table if user-facing.
5. Commit, tag, push (see Release process below).

## Modifying an existing skill

1. Edit `skills/<name>/SKILL.md` directly.
2. If the change is user-visible (new triggers, behavioral change), bump `version` in BOTH manifests.
3. If it's a pure wording polish or doc cleanup, no version bump required.

## Release process

```bash
# 1. Bump version in both manifests (keep them identical)
#    .claude-plugin/plugin.json
#    .codex-plugin/plugin.json

# 2. Commit
git add -A
git commit -m "Release v<X.Y.Z>: <summary>"

# 3. Tag
git tag v<X.Y.Z>

# 4. Push
git push && git push --tags
```

Both Claude Code and Codex clients pick up the new version from the tag (Claude Code auto-updates on startup; Codex behavior on auto-update is undocumented as of this writing — users may need to run `codex marketplace update qa-vault` manually).

## Companion marketplace

This plugin is listed in the `qa-vault/marketplace` catalog. When renaming the plugin or changing the GitHub repo path, update the entry in that repo's `.claude-plugin/marketplace.json` AND `.agents/plugins/marketplace.json`.

## What this plugin does NOT contain

- Slash commands (`commands/`)
- Subagents (`agents/`)
- Hooks (`hooks/`)
- MCP servers (`.mcp.json`)

It is a pure skills plugin. If any of the above are added in the future, they must be declared in both manifests if both tools support them, or the plugin scope must be documented as ecosystem-specific.
