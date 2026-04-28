# Default Implementation Doc Template

Use this template when the project has no existing documentation conventions. If existing
docs are present, match their format instead — this template is a fallback, not a mandate.

Adapt the level of detail to the complexity of the feature. Simple features should produce
short docs. Complex systems can use all sections and add subsections as needed.

## Template

```markdown
---
# Required. Kebab-case identifier. Should match the filename (without .md).
name: <feature-or-module-name>
# Required. One-line hook for AI triage. Specific and action-oriented; the router uses
# this to decide whether to load the doc into the agent's context.
description: <one sentence describing what this doc covers and why an agent would read it>
# Optional. Keywords/verbs that should bias the router toward loading this doc.
# triggers: [auth, login, jwt, session]
# Optional. Names (matching the `name` field) of related docs.
# related: [api-gateway, session-store]
---

# <Feature / Module Name>

## Overview

<One to three sentences. What this does and why it exists. An agent reading only this
paragraph should know whether this doc is relevant to its current task.>

## Problem

<What problem or need prompted this implementation. Include enough context that a reader
with no prior knowledge understands the motivation. Omit this section for features where
the need is self-evident.>

## Approach

<How the implementation works. This is the meat of the document.>

<Describe the architecture, key components, data flow, and important code paths. The
level of detail should match the complexity — a simple utility gets a paragraph or two,
a complex subsystem might need multiple subsections with component descriptions.>

<If helpful, describe the flow in sequence:>
<1. Request arrives at X>
<2. X validates and passes to Y>
<3. Y does Z>
<4. Result returned via W>

<For complex features, break into subsections:>

### <Subcomponent A>

<How this part works.>

### <Subcomponent B>

<How this part works.>

## Trade-offs & Constraints

<What alternatives were considered and why this approach won. Be specific — naming the
rejected alternatives and their downsides helps future developers understand the design
space.>

<What assumptions the current implementation makes. What would break if those assumptions
changed? What are the known limitations?>

<Example:>
<- Chose in-memory token store over Redis for simplicity. This means the auth state is
  lost on restart and the service cannot be horizontally scaled without adding a shared
  store.>
<- Rate limiting is per-process, not distributed. If running multiple instances behind
  a load balancer, each instance tracks limits independently.>

## Key Files

| File | Purpose |
|------|---------|
| `path/to/file.py` | <What this file does in the context of this feature> |
| `path/to/other.py` | <What this file does> |

## Things to Know

<Non-obvious behaviors, gotchas, edge cases, configuration dependencies, performance
notes, or anything a developer needs to be aware of before modifying this code.>

<Example:>
<- The `validate_token()` function silently returns `None` on any error rather than
  raising — callers must check the return value, not catch exceptions.>
<- The migration script must run before the API starts; there's no runtime migration.>
<- Processing time scales linearly with batch size; batches over 10K items may cause
  timeouts with the default 30s limit.>

## Related Docs

<Links or references to other relevant documentation. Only include this section if
related docs actually exist in the project.>

<- `docs/api-gateway.md` — describes the gateway layer that routes to this service>
<- `docs/database-schema.md` — schema definitions used by this module>
```

## Section Usage Guide

| Section              | When to include                                         |
|----------------------|---------------------------------------------------------|
| Overview             | Always                                                  |
| Problem              | When the motivation isn't obvious from the feature name |
| Approach             | Always                                                  |
| Trade-offs           | When design choices were made that have consequences    |
| Key Files            | When more than 2-3 files are involved                   |
| Things to Know       | When there are gotchas or non-obvious behaviors         |
| Related Docs         | When related documentation exists in the project        |
