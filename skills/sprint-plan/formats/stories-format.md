# Preamble

This file is the format of `.stories/STORIES.md`, the append-only user story log. `/sprint-plan` writes it; a sprint plan, a ticket and a commit cite a story by its ID.

## Gate

A want earns a `US-###` when all conditions are met:

1. **Applies to a user** — a person the product is built for.
2. **Scoped to a user role** — a class of user the system tells apart.
3. **Fits the user role** — a user in that role could ask for it.
4. **Experienced as an outcome** — a result the user can point at, with a clear boundary around what satisfies the want.
5. **Carries a motive** — a reader learns why the user wants it.

## File

- **Location:** `.stories/STORIES.md` is one file, with every record, appended in creation order.
- **Shape:** The first line is the first record's `## US-001` heading. Every record is a `## US-###` block. A parser uses `## ` to find each record. The file has no title, no intro, no separators between records.
- **Numbering:** `US-###` numbers are sequential and zero-padded to three digits (`US-001`, `US-002`, …). The numbers are unique, have no gap, and never appear out of order.

## Template

```markdown
## US-### — <title>

[- **Status:** proposed | superseded | deprecated]
[- **Supersedes:** US-###]
- **Date Created:** YYYY-MM-DD
[- **Date Last Edited:** YYYY-MM-DD]

> <narrative: As a <user role>, [when <situation>, ]I want <outcome>, so that <motive>.>

- **User role:** <user role>[, <user role>]
- **Source:** external user | internal user | developer
[- **Related to:** <ID>[, <ID>]]
```

One entry per slot:

| Slot | Rule |
|---|---|
| **Status** | Optional, single line. Added once, with `Date Last Edited`. The closed value set is `proposed`, `superseded`, or `deprecated`. Absence is not missing data: by definition, absence means `accepted`. |
| **Supersedes** | Optional, single line. `US-###` is the story this record replaces. Never edited. |
| **Date Created** | Mandatory. `YYYY-MM-DD` is the day the story was recorded. Never edited. |
| **Date Last Edited** | Optional. `YYYY-MM-DD` is the day of the last change to the record. Absent until then. |
| **Narrative** | Mandatory. One sentence, as the blockquote: `As a <user role>, [when <situation>, ]I want <outcome>, so that <motive>.` The `when` clause is optional. It names the moment the want begins. |
| **User role** | Mandatory. One or more, from `.brief/brief.md#user-roles`. Edited as the roles evolve. |
| **Source** | Mandatory, single line. The closed value set is `external user`, `internal user`, or `developer`: who came up with the want. |
| **Related to** | Optional. IDs only: `US-###`, `ADR-###`, comma-separated, in that order when mixed. An edge on the map: written on either record, it connects both. Every ID resolves to a heading. Edited as the system grows. |

## Mutability

- **User role** is edited as the roles evolve, and **Related to** as the system grows, with `Date Last Edited`. Nothing else is.
- **Status** is added once, to replace or deprecate a story, with `Date Last Edited`. To replace, add a **new** record with `Supersedes`, then mark the old one `superseded`.
- **Everything else is frozen.** Never change or remove it.

## Ownership

`.stories/STORIES.md` is the source of truth. Artifacts reference a record by `US-###`. Nobody quotes a record. Whether a story is built is a GitHub fact, never a line here.

## Example

```markdown
## US-014 — Paid orders without a manual check

- **Date Created:** 2026-10-02

> As a shop owner, when a customer pays at checkout, I want the order to show as paid, so that I can ship it without checking Stripe by hand.

- **User role:** shop owner
- **Source:** external user
- **Related to:** ADR-009
```
