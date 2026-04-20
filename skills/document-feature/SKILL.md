---
name: document-feature
description: >
  Create and maintain implementation documentation that describes how features work, why they
  were built the way they were, and what to watch out for when modifying them. Use this skill
  whenever the user asks to "document what was implemented", "write up how this works",
  "document this feature", "update the docs for this", or any request to produce or update
  technical documentation about implemented functionality. Also trigger when the user finishes
  implementing a feature or making significant changes and asks to wrap up — suggest
  documentation if it hasn't been requested. This skill does NOT produce changelogs, session
  logs, or decision records. It produces living reference documents that explain the approach,
  the problem it solves, trade-offs, and what a future developer (human or AI) needs to know
  to safely modify the functionality.
---

# Implementation Documentation Skill

Create and maintain reference documentation for implemented features and functionality.
Each document answers the questions a developer (human or AI) would ask before touching
the code: What does this do? Why was it done this way? What are the gotchas?

This is NOT a changelog or session log. It doesn't track *when* things happened or *who*
did them. It describes *how things work right now* and *why*.

## What Makes a Good Implementation Doc

Think of it like a briefing document. An AI agent picks up a task that says "add rate
limiting to the auth endpoints." Before writing a line of code, it reads the auth
implementation doc and learns: the auth module uses JWT with symmetric HS256, tokens are
verified via middleware on each request, the token store is in-memory (no Redis yet), and
the original design chose simplicity over horizontal scalability. Now the agent can make
an informed decision about where and how to add rate limiting without breaking assumptions.

A good implementation doc has these qualities:

- **Self-contained.** A reader with zero prior context can understand the feature.
- **Honest about trade-offs.** Documents what was chosen AND what was sacrificed.
- **Actionable.** Contains enough detail that someone can modify the feature safely.
- **Current.** Reflects the code as it exists right now, not a historical record.
- **Concise.** Covers what matters, skips what doesn't. No padding.

## Step 0: Discover Existing Documentation

Before writing anything, scan the project for existing docs. The goal is consistency —
new documents should look like they belong.

### Discovery Procedure

1. **Find the docs directory.** Look for `docs/`, `doc/`, `documentation/`, or docs at
   the project root. Check `README.md` for links to documentation. If nothing exists,
   ask the user where docs should live.

2. **Study existing docs.** If implementation docs already exist, open 2–3 of them and
   extract the conventions:
   - File naming pattern (kebab-case? UPPERCASE? numbered?)
   - Document structure (what sections do they use? what order?)
   - Frontmatter (YAML? which fields?)
   - Formatting style (prose vs bullets, heading levels, code reference style)
   - Level of detail (high-level overviews vs deep implementation details?)
   - How they cross-reference other docs or code

3. **Match those conventions.** Every new doc must feel like it was written by the same
   author as the existing ones. If the project uses terse bullet-point style, don't write
   paragraphs. If docs use YAML frontmatter with specific fields, use the same fields.

### When No Docs Exist

If the project has no documentation yet, use the default structure described below. But
still look at the codebase for style cues — if the README uses a certain heading style
or naming convention, carry that forward.

Default location: `docs/` at project root.
Default naming: `kebab-case.md` (e.g., `auth-module.md`, `payment-processing.md`).

## Document Structure

Use this structure as the default when no existing convention is found. If the project
already has implementation docs, match their structure instead.

Read `references/doc_template.md` for the full default template.

The core sections are:

1. **Overview** — One to three sentences. What this feature/module does at a high level.
   An AI agent should be able to read just this and know if this doc is relevant to its
   current task.

2. **Problem** — What problem or need this implementation addresses. Include enough
   context that the "why" is obvious. Skip this section if the feature is self-evidently
   necessary (e.g., "user login" doesn't need a problem statement, but "custom event
   batching queue" probably does).

3. **Approach** — How the implementation works. This is the core of the document.
   Describe the architecture, the key components, how data flows, what the important
   code paths are. Use whatever level of detail is appropriate — a simple utility
   function needs a paragraph, a complex subsystem might need diagrams described in
   text, component breakdowns, and sequence descriptions.

4. **Trade-offs & Constraints** — What alternatives existed and why this approach was
   chosen over them. What limitations does the current implementation have? What
   assumptions does it make? This section is critical because it tells a future
   developer what they *cannot* safely change without rethinking the design.

5. **Key Files** — A table or list mapping the important files/modules to what they do.
   Paths should be relative to project root. Don't list every file — focus on the ones
   someone would need to understand or modify.

6. **Things to Know** — Gotchas, non-obvious behaviors, edge cases, configuration
   dependencies, performance characteristics, or anything that would surprise someone
   working on this code for the first time. Think of this as "what I wish someone told
   me before I started modifying this."

7. **Related Docs** — Links or references to other documentation that provides context.
   Only include if relevant docs actually exist.

Not every document needs all sections. A small utility might only need Overview, Approach,
and Key Files. A complex system might need all of them plus additional subsections. Use
judgment.

## Workflow

### Creating a New Document

1. Run Step 0 (discover existing docs).
2. Identify what was implemented — review the conversation, code changes, or ask the user.
3. Determine the scope — is this one doc or multiple? A new auth system might warrant one
   doc for the whole module; a large platform might need separate docs per component.
4. Check if a relevant doc already exists — if so, update it instead of creating a new one.
5. Write the doc following existing conventions or the default template.
6. Present a brief summary to the user and offer to adjust.

### Updating an Existing Document

When functionality covered by an existing doc has been changed:

1. Run Step 0 — re-read the existing doc and the project's conventions.
2. Identify what changed and which sections are affected.
3. Update the affected sections to reflect the current state. Implementation docs are
   living documents — they describe how things work *now*, not how they used to work.
   Don't append "Update: we changed X" — just rewrite the section to be accurate.
4. If the changes are significant enough that trade-offs or constraints have shifted,
   update those sections too.
5. If the scope of the feature has grown such that the existing doc structure doesn't
   fit anymore, restructure it. It's fine to reorganize sections, split a doc into
   multiple docs, or merge docs — as long as the result is clear and accurate.

### Deciding What to Document

Not everything needs a dedicated implementation doc. Use this as a guide:

**Probably needs a doc:**
- A new module, service, or subsystem
- A complex feature with non-obvious behavior
- Anything involving trade-offs that aren't visible in the code
- Infrastructure or configuration that requires explanation
- Integration points with external services

**Probably doesn't need its own doc (mention in a parent doc instead):**
- A simple bug fix
- A straightforward CRUD endpoint following established patterns
- A minor configuration change
- Code that is self-documenting and follows obvious patterns

## AI-Readability Principles

These docs are consumed by AI agents first. That means:

- **Be explicit.** Don't rely on context from a conversation. State facts directly.
- **Name things consistently.** Always use the exact same string to refer to a file,
  function, or component. If the codebase calls it `AuthMiddleware`, don't also call
  it "the auth layer" or "authentication middleware" — use `AuthMiddleware`.
- **Frontload the important information.** The first sentence of each section should
  give an agent enough to decide if it needs to read further.
- **Use file paths from project root.** Always relative, always exact.
- **Keep it current.** An outdated doc is worse than no doc. When updating functionality,
  update the doc to match. Remove information that is no longer true.
