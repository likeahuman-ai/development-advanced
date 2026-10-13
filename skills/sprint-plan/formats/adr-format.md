# Preamble

This file is the format of `.adr/ADR.md`, the append-only architectural decision log. `/sprint-plan` writes it. Tickets cite a decision on their `Related to` line; every later phase treats a decision as standing law.

## Gate

A decision earns an `ADR-###` when all conditions are met:

1. **Hard to reverse** — changing your mind later costs something real.
2. **Surprising without context** — a future reader will wonder "why did they do it this way?"
3. **A real trade-off** — genuine alternatives existed and one was picked for specific reasons.
4. **Shapes the system** — it defines how the parts form the whole.

## File

- **Location:** `.adr/ADR.md` is one file, with every record, appended in creation order.
- **Shape:** The first line is the first record's `## ADR-001` heading. Every record is an `## ADR-###` block. A parser uses `## ` to find each record. The file has no title, no intro, no separators between records.
- **Numbering:** `ADR-###` numbers are sequential and zero-padded to three digits (`ADR-001`, `ADR-002`, …). The numbers are unique, have no gap, and never appear out of order.

## Template

```markdown
## ADR-### — <title>

[- **Status:** proposed | superseded | deprecated]
[- **Supersedes:** ADR-###]
- **Date Created:** YYYY-MM-DD
[- **Date Last Edited:** YYYY-MM-DD]

> <Y-statement: context · concern · decision · goal · trade-off>

- **Decision:** <what was chosen, declarative>
- **Alternatives rejected:**
  - <alternative actually weighed> — <one-line reason>
- **Locks in:** <forward constraints this creates>
- **Makes harder:** <costs / options now ruled out>
- **Testable:** no | yes — <trigger>
[- **Related to:** <ID>[, <ID>]]
```

One entry per slot:

| Slot | Rule |
|---|---|
| **Status** | Optional, single line. Added once, with `Date Last Edited`. The closed value set is `proposed`, `superseded`, or `deprecated`. Absence is not missing data: by definition, absence means `accepted`. |
| **Supersedes** | Optional, single line. `ADR-###` is the decision this record replaces. Never edited. |
| **Date Created** | Mandatory. `YYYY-MM-DD` is the day the decision was recorded. Never edited. |
| **Date Last Edited** | Optional. `YYYY-MM-DD` is the day of the last change to the record. Absent until then. |
| **Y-statement** | Mandatory. One sentence, as the blockquote: `In the context of <context>, facing <concern>, we decided <decision> to achieve <goal>, accepting <trade-off>.` It stands alone: an agent that reads only this line knows the decision. |
| **Decision** | Mandatory. State the decision in a declarative sentence. |
| **Alternatives rejected** | Mandatory. List **only the alternatives you genuinely weighed**, each with a one-line reason. Never invent options to reach a count. |
| **Locks in** | Mandatory. The forward constraints this decision creates for later work. |
| **Makes harder** | Mandatory. The cost: the options this decision rules out or makes expensive. |
| **Testable** | Mandatory. `no`, or `yes` with a trigger that would reopen it (for example, "provider exposes X" or "load exceeds Y"). |
| **Related to** | Optional. IDs only: `US-###`, `ADR-###`, comma-separated, in that order when mixed. An edge on the map: written on either record, it connects both. Every ID resolves to a heading. Edited as the system grows. |

## Mutability

- **Related to** is edited as the system grows, with `Date Last Edited`. Nothing else is.
- **Status** is added once, to replace or deprecate a decision, with `Date Last Edited`. To replace, add a **new** record with `Supersedes`, then mark the old one `superseded`.
- **Everything else is frozen.** Never change or remove it.

## Ownership

`.adr/ADR.md` is the source of truth. Artifacts reference a record by `ADR-###`. Nobody quotes a record, except an agent prompt that lifts a Y-statement as a constraint.

## Example

```markdown
## ADR-009 — Webhooks verify the signature before they read the body

- **Date Created:** 2026-10-02

> In the context of Stripe and Luma webhooks that change order state, facing forged requests, we decided that every handler verifies the signature with the shared `verifyStripeSig` helper before it reads the body, to achieve one tested way in, accepting that a handler cannot log an unsigned body.

- **Decision:** Every webhook handler calls the shared signature helper first and returns 400 on failure.
- **Alternatives rejected:**
  - Verify inside each handler by hand — the check drifts per handler.
  - Verify at the router — the router stays pure, with no logic.
- **Locks in:** a new webhook handler copies `convex/lumaWebhook.ts`.
- **Makes harder:** logging the raw body of a rejected request.
- **Testable:** yes — a provider sends unsigned events by design.
- **Related to:** US-014
```
