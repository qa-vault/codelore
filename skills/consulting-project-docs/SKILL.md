---
name: consulting-project-docs
description: >
  Project-documentation router. Use BEFORE any planning, investigation, debugging, or
  implementation task — whenever the user asks to "plan X", "investigate Y", "figure out
  why Z", "debug this", "fix this bug", "add a feature to", "explore how X works", "why
  does", "how does", "onboard onto this code", or otherwise needs to understand existing
  functionality before acting. Reads `docs/INDEX.md` (auto-maintained by the codelore
  `document-feature` skill) and pulls only the implementation docs whose `description` or
  `triggers` match the current task into the agent's working context, so the agent
  doesn't plan or fix in the dark. If `docs/INDEX.md` is missing in a project that
  already has `docs/`, suggests running the `migrate-project-docs` skill before
  continuing. This is NOT for writing docs (that's `document-feature`) and NOT for
  exploratory critique (that's `exploratory-qa`) — it is for surfacing the right
  existing docs at the right moment.
---

# Consulting Project Docs

This skill is the front door for AI sessions that need context from a project's existing
implementation documentation. Without it, agents plan and investigate without consulting
the docs that contain critical context — leading to hallucinated assumptions and rework.

## When to invoke

Trigger on planning/investigation/debugging verbs the user uses **before** asking the
agent to act on existing functionality. Concrete signals:

- "How does X work?" / "Why does Y do Z?"
- "Plan a change to ..." / "Investigate ..." / "Fix the bug in ..."
- "Add Feature B to the existing X module" — the X module is the part you need to load
  docs for, not the new feature.
- "Onboard me onto this codebase" / "Explain this module"
- The agent is about to call `EnterPlanMode`, debug a non-trivial bug, or modify a
  module it has not touched in this session.

Skip invocation for:

- Trivial edits (renames, formatting, single-line bug fixes in code you've just read).
- Pure greenfield work where no existing module is involved.
- Tasks the user has already loaded context for in this conversation.

## Workflow

### 1. Locate `docs/INDEX.md`

Look at the project root for `docs/INDEX.md`. In monorepos, also check one level deeper
(e.g. `packages/*/docs/INDEX.md`, `apps/*/docs/INDEX.md`) and prefer the index closest
to the files the user is asking about.

If no `INDEX.md` is found anywhere, also check whether `docs/` (or `doc/`,
`documentation/`) exists with `.md` files in it.

### 2. Branch on index state

| State | Action |
|-------|--------|
| `INDEX.md` found and recent | Go to step 3 (dispatch). |
| No `INDEX.md`, but `docs/` has `.md` files (excluding plan folders) | Tell the user codelore can index these for AI sessions and offer to invoke the `migrate-project-docs` skill. Pause until the user accepts or declines. If declined, defer back without loading anything — do not silently grep individual docs. |
| No `INDEX.md`, no `docs/` (or only plan folders) | Defer back silently. There is nothing to consult; let the agent proceed with normal exploration tools. |

### 3. Dispatch — pick docs to load

Read `docs/INDEX.md` in full. For every entry, judge whether the doc's `description`
(and `triggers` if present) is relevant to the current task. Be selective — loading
every doc defeats the purpose.

Heuristics:

- Match on the noun/module the user mentioned (e.g. "auth", "payments", "queue").
- Include cross-referenced docs via the `related` field of each loaded doc, but only
  one level deep — do not transitively expand.
- Cap at ~3–5 docs unless the task is genuinely cross-cutting. If you'd load more,
  ask the user which area they want to focus on first.

Read the selected docs in full. Surface a one-line note to the user listing which docs
were loaded and why, so they can correct the selection.

### 4. Stale-check (delta migration nudge)

After dispatching, do one quick consistency check:

- List `.md` files under `docs/` (excluding `INDEX.md`, `docs/plans/`, `plans/`, `specs/`).
- Cross-reference against the entries in `INDEX.md`.
- If any files are missing from the index OR have no YAML frontmatter, mention them by
  name and suggest running `migrate-project-docs` for those specific files. Do not
  block the current task — this is a nudge, not a gate.

If everything is in sync, say nothing about migration.

## Plans are off-limits

Never load files under `docs/plans/`, `plans/`, or `specs/`. These are pre-implementation
proposals, not reference docs — including them would mislead the agent about what's
actually implemented. If the user explicitly asks to consult a plan, that's a separate
request handled by normal file-reading tools, not by this router.

## What this skill does NOT do

- It does not write or modify documentation. That is `document-feature`.
- It does not critique or QA an implementation. That is `exploratory-qa`.
- It does not bulk-add frontmatter to legacy docs. That is `migrate-project-docs`.
- It does not run git, search the web, or invoke MCP tools — it is a pure
  read-and-route skill.
