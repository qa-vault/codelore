---
name: migrate-project-docs
description: >
  One-shot bulk migration that prepares a project's existing documentation for the
  codelore router (`consulting-project-docs`). Use when the user asks to "migrate the
  docs", "set up codelore docs", "index existing documentation", "add frontmatter to
  the docs", or when `consulting-project-docs` detects that `docs/INDEX.md` is missing
  in a project that already has `docs/`. Walks `docs/`, classifies each `.md` file as
  an implementation doc, a plan/spec, or ambiguous, asks the user about the ambiguous
  ones, then bulk-adds YAML frontmatter to approved implementation docs and writes
  `docs/INDEX.md`. Idempotent — re-running on a fully migrated tree is a no-op. Skips
  silently if there is nothing to migrate. Does NOT touch files under `docs/plans/`,
  `plans/`, or `specs/`. This is a one-time setup skill per project (with delta
  migrations for newly added unfrontmattered docs); it is not for ongoing doc
  authoring — that is `document-feature`.
---

# Migrate Project Docs

Bulk-prepares a project's existing markdown documentation for the codelore router
skill (`consulting-project-docs`) by adding YAML frontmatter to implementation docs and
generating `docs/INDEX.md`.

The router is useless on day one in a pre-existing project: `docs/INDEX.md` doesn't
exist and old docs have no frontmatter, so there is nothing to dispatch from. This
skill closes that gap.

## When to invoke

- The user explicitly asks: "migrate the docs", "set up codelore docs", "index
  existing documentation", "add frontmatter to docs".
- The `consulting-project-docs` skill detected a missing `INDEX.md` and the user
  agreed to migration.
- The `consulting-project-docs` skill detected new unfrontmattered docs after a
  previous migration, and the user agreed to a delta migration.

Do not auto-invoke. The user must agree before any file is touched — this skill
modifies many files at once.

## Frontmatter contract

Every approved doc gets exactly this block prepended (no other edits to the body):

```yaml
---
name: <kebab-case identifier, derived from filename without .md>
description: <one-line hook describing the doc, written for AI triage>
---
```

`triggers` and `related` are left unset — the user can fill them in later. See the
`document-feature` skill for the full schema and field rules.

## Workflow

### 1. Discover candidates

Walk the project's documentation root (`docs/` by default; if not present, ask the
user). Collect every `.md` file under it.

**Exclude unconditionally:**

- `docs/INDEX.md` itself.
- Anything under `docs/plans/`, `plans/`, or `specs/` (these are plans, not impl docs).
- `README.md` if it lives at `docs/README.md` and is a navigation index rather than
  a feature doc — use judgment; if unsure, ask.
- Files that already start with a valid YAML frontmatter block containing both `name`
  and `description`. These are already migrated.

If the resulting candidate list is empty, **stop here**. Report "nothing to migrate"
and exit without touching any file.

### 2. Classify candidates

For each candidate, classify:

| Class | Signals |
|-------|---------|
| **Implementation doc (confident)** | Located at `docs/<name>.md`, content describes how an implemented feature/module works, no plan-style language ("we will", "the proposed approach", "open questions"). |
| **Plan / spec (confident)** | Filename matches `*-plan.md`, `*-proposal.md`, `*-spec.md`, `*-rfc.md`. Or first heading reads like a proposal ("RFC: ...", "Plan: ..."). Or content is dominated by future-tense / "open questions" sections. |
| **Ambiguous** | Anything that isn't clearly one or the other. |

Confident impl docs go into the migration queue. Confident plans are excluded entirely
(do not move them, do not modify them — just leave them alone). Ambiguous files go
into a confirmation queue.

### 3. Confirm with the user

Show the user three lists:

1. **Will migrate** (confident impl docs). Offer batch approve / batch skip / per-file.
2. **Need your call** (ambiguous). For each, show filename + first heading + a 1–2
   line excerpt. Ask: migrate / skip. Allow batch decisions.
3. **Excluded as plans** (confident plans). Inform only — these will not be touched.

Wait for user confirmation. Do not write any file before this step.

### 4. Generate frontmatter for approved files

For each approved file:

- Derive `name` from the filename (strip `.md`, ensure kebab-case).
- Derive `description` from the doc itself:
  - Prefer the doc's `## Overview` section if it has one.
  - Otherwise use the first non-heading paragraph.
  - Reduce to a single sentence (≤ 200 chars). The description must be specific
    enough that the router can decide relevance — never use generic phrases like
    "Documentation for X" or "About the X module".
  - If you cannot extract a usable description, ask the user for one. Do not invent.
- Prepend the frontmatter block to the file. Preserve everything else byte-for-byte.

### 5. Generate or refresh `docs/INDEX.md`

After all approved files are updated, regenerate `docs/INDEX.md` per the format
specified in the `document-feature` skill's "Index Maintenance" section:

```markdown
# Project Documentation Index

This file is auto-maintained by the codelore `document-feature` and
`migrate-project-docs` skills. Do not hand-edit — re-run the appropriate skill instead.

| Name | Description | File |
|------|-------------|------|
| <name> | <description> | [<filename>](<filename>) |
...
```

Sort alphabetically by `name`. List every doc that has valid frontmatter — including
docs that were already migrated before this run.

### 6. Report back

Tell the user:

- How many docs got frontmatter.
- How many were skipped (and why — plan, ambiguous-declined, already migrated).
- The path to the (new or updated) `INDEX.md`.

Keep it under ten lines. The user can open `INDEX.md` to see the full list.

## Idempotence

A second run of this skill on a fully migrated tree must:

1. Discover candidates → finds none (every impl doc already has frontmatter).
2. Hit the early-exit at the end of step 1.
3. Touch no files.
4. Report "nothing to migrate".

This makes it safe for the router to suggest migration whenever it sees any
unfrontmattered file, without risk of clobbering already-migrated content.

## Delta mode

When invoked because of a stale-check from `consulting-project-docs` (new
unfrontmattered file added after a prior migration), the workflow is unchanged — the
discovery step naturally narrows to just those files because everything else already
has frontmatter and is excluded by step 1.

## Hard rules

- Never write or modify files under `docs/plans/`, `plans/`, or `specs/`.
- Never modify a file's body — only prepend frontmatter to approved files.
- Never create an empty `INDEX.md`.
- Never proceed past step 3 without explicit user confirmation.
- Never invent a `description` — ask the user when the doc itself doesn't yield one.
