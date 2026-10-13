# Preamble

This file is the format of `.spec/spec.md`, the system specification: how the system is built in the code on `main`. It is a claim about code, so the code wins where the two disagree. `spec-writer` patches it once per sprint, after the sprint has landed: the next `/sprint-plan` run commits the patch in its plan PR. Every phase reads the slices it needs.

## Gate

A fact earns a line when all conditions are met:

1. **True of the landed code** — it describes what is on `main` now, never what is planned.
2. **Shape, not detail** — it is at the level of components, flows, data, contracts and patterns, above the implementation of one function.
3. **Owned nowhere else** — a decision lives in an `ADR-###`, a want in a `US-###`, a test in the code. The spec points at them.

## File

- **Location:** `.spec/spec.md` is one file. A section moves to `.spec/<section>.md` only when it outgrows the file; its anchor stays the same.
- **Shape:** YAML frontmatter, then the nine `## ` sections of the template, in template order, all present. A section with nothing built yet reads `None yet.` A `### ` heading appears only under `## Crosscutting Concepts & Patterns`, one per concept. The other sections are flat lists.
- **Numbering:** None. A heading's slug is its anchor: `.spec/spec.md#api-surface`. A heading's text never changes, because tickets and plans point at it.

## Template

```markdown
---
last_updated: YYYY-MM-DD
sprint: v{N}
---

## Architecture
## Runtime / Data-flow view
## Data Model
## API Surface
## Crosscutting Concepts & Patterns
## Stack
## Directory pointer-map
## Infrastructure
## Constraints
```

One entry per slot:

| Slot | Rule |
|---|---|
| **last_updated · sprint** | Mandatory. The day and the sprint of the last patch, so a reader can tell when the spec last matched the code. |
| **Architecture** | The parts and how they relate: one line per component with its role, then a list of relations. |
| **Runtime / Data-flow view** | How a request or an event moves through the parts, as ordered steps, `A → B → C`. Live paths only. |
| **Data Model** | The persistent shapes: entity, fields, types; relations as a list. |
| **API Surface** | The callable contract: one line per endpoint or signature, with its purpose and its file. A long list can group under a bold label, never a `### ` heading. |
| **Crosscutting Concepts & Patterns** | One `### ` per system-wide pattern (auth, errors, idempotency, logging): the pattern once, then the parts that follow it. |
| **Stack** | Languages, runtimes, frameworks and services actually in use, each with its role; a version only where it constrains. A choice cites its `ADR-###`. |
| **Directory pointer-map** | `path/ → role`, one line per part. |
| **Infrastructure** | The runtime substrate as built: queues, datastores, schedulers, each with its role. |
| **Constraints** | Standing limits the code is built around, one per line. |

## Mutability

- **Created once, then patched.** The first sprint writes all nine sections. After that, a patch touches only the sections the sprint's code changed, and leaves the rest byte for byte.
- **Remove what the code removed.** Deleted code is deleted from the spec in the same patch. A section emptied this way reads `None yet.`
- **Drift is fixed, not hidden.** A wrong line the sprint did not cause is fixed in the same patch, and the plan PR's Summary names it.
- **No history in the file.** `git log -p .spec/spec.md` is the history.

## Ownership

The code is the source of truth; the spec is the map of it. Artifacts reference a section by its anchor. The spec cites `ADR-###`, `US-###` and file paths, and never copies them. Nobody quotes the spec.

## Example

A patched excerpt:

```markdown
---
last_updated: 2026-10-14
sprint: v7
---

## API Surface
- POST `/stripe/webhook` → verify the signature, mark the order paid (`convex/stripeWebhook.ts`)

## Crosscutting Concepts & Patterns

### Idempotency
A replayed event finds its row already in the target state and writes nothing (ADR-009). Followed by: `convex/stripeWebhook.ts`, `convex/lumaWebhook.ts`.
```
