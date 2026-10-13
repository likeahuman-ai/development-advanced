# Preamble

This file is the format of a ticket: a GitHub issue with the `ticket` label. One buildable outcome of the sprint, landed as one atomic commit in one pull request. The body is the work order an implementer executes and a reviewer accepts. `/sprint-tickets` writes it; `/sprint-build` and `/sprint-review` read it.

## Gate

A draft survives as a single ticket when all conditions are met:

1. **Design and scope are settled** — the intended behaviour, the boundaries and the governing decisions are explicit; implementation details can be resolved during the build.
2. **The affected code is identified** — the files and interfaces known before the build are named; a file the build discovers is added, not a reason to stop.
3. **Failure and success are demonstrable** — a test exposes the gap at the base commit and passes after the change.
4. **One complete outcome** — the ticket owns the whole change, every affected file, through to the passing test, no stubs. Related work stays together however large the change. It splits only into outcomes that are each independent, and buildable and testable on their own.
5. **Under an epic** — it advances one epic of the sprint plan.

## File

- **Location:** A GitHub issue, one per ticket, with the labels `ticket`, `v{N}` and the `epic:<name>` of its epic, and its blockers linked as GitHub's blocked-by dependency. Nothing lives in a dotdir.
- **Shape:** The body is flat `## ` sections, in template order, all present, no prose before the first. A parser uses `## ` to find each section. No section outside the template, no nested headings.
- **Numbering:** `#N` is GitHub's number. A ticket is created after the tickets that block it, so every `#N` it names exists and none is invented. Acceptance criteria are `AC-n`, sequential from `AC-1` in emission order. A number is never reused and never renumbered: a struck criterion's gap stays (`AC-1 · AC-3` is a conformant body). Outside the issue a criterion is `#N AC-n`.
- **Branch:** The ticket builds on `ticket/<N>-<slug>`, the slug taken from its title.

## Template

```markdown
## Objective
<one sentence: what this change is>

- **Blocked by:** #N[ #N] | —
- **Related to:** US-###[, ADR-###] | —

## Context
- <file:line> — <why it is relevant>
- new: <path> — <what it is>
- test: <path or command> — <the test that fails today and passes after the change>
- pattern: <path> — <the file whose shape this change copies>
[- spec: .spec/spec.md#<anchor> — <the documented behaviour it touches>]

## Change
<the decided shape: the functions, tables, routes or events it adds or alters, in fences. The behaviour in each case, the error cases included, in prose. Never code.>
- Stays as it is: <what this change must not touch or alter>

## Acceptance Criteria
- AC-1: Given <state>, When <action>, Then <observable outcome>
```

One entry per slot:

| Slot | Rule |
|---|---|
| **Objective** | Mandatory. One sentence: what this change is, scope and intent. The implementer receives it as the task. |
| **Blocked by** | Mandatory, single line. The upstream tickets this one consumes, `#N` space-separated, `—` when none. GitHub's blocked-by dependency carries the same tickets, and `/sprint-build` reads GitHub. No `Blocks:` line: the edge lives on the downstream ticket. |
| **Related to** | Mandatory, single line. The stories it serves, then the decisions that govern it: `US-###`, `ADR-###`, comma-separated, in that order. `—` when none. Cite the ID, never the text. The ticket's commit copies them to its `Story:` and `ADR:` trailers. |
| **Context** | Mandatory. One line per touched location, each with a fixed prefix. An existing file is `file:line`, the seam where the change lands. A file this ticket creates is `new: <path>`, no line. `test:` names the one test that exposes the gap today and verifies the change after: existing, or new and written by the implementer from the criteria. `pattern:` is optional. `spec:` is present when a section of `.spec/spec.md` describes the code it touches. When the change builds against the output of a tool the implementer cannot run, a verbatim sample of that output goes here in a fence. |
| **Change** | Mandatory. The shape in fences: signatures, columns, routes, event types. The behaviour in prose, one case at a time, the error cases included. Never code: a loop, a helper name or pseudo-code is read as production code. Every design and scope decision is made here or in the decisions on `Related to`; no "or", "either" or "TBD" remains. `Stays as it is:` closes it, one line, mandatory. |
| **Acceptance Criteria** | Mandatory. Numbered `AC-n:`. Given/When/Then, unless a criterion depends on prior state or a count; then EARS: `When <trigger>, the <system> shall <response>`. One notation per ticket. Every criterion is the plain-language twin of the `test:` check; the test decides. A criterion names only names that Change gives. Never restates a project-wide check: those live in the build-order's `## Verify`. |

## Mutability

- **Acceptance Criteria** are appended, never renumbered. A struck criterion keeps its number as a gap.
- **Blocked by** is edited when a blocker appears later, and GitHub's dependency follows it.
- **Everything else is frozen** once the ticket is dispatched. A change of Objective is a new ticket.

## Ownership

GitHub is the source of truth. Artifacts reference a ticket by `#N` and a criterion by `#N AC-n`. Nobody quotes a ticket. A ticket lands as one commit, and the commit names it with `Closes:`. `Blocked by` and GitHub's blocked-by dependency name the same tickets, and a ticket where they differ is a defect. The `epic:` label names an epic of the sprint plan, and a ticket without one is a defect.

## Example

```markdown
## Objective
Add a Stripe webhook handler that marks an order paid on `checkout.session.completed`.

- **Blocked by:** #104
- **Related to:** US-014, ADR-009

## Context
- convex/http.ts:48 — HTTP router; webhook routes register here
- convex/orders.ts:96 — `orders` table and status enum
- new: convex/stripeWebhook.ts — the handler module this ticket adds
- test: convex/stripeWebhook.test.ts — replay and unmatched-event cases, red today
- pattern: convex/lumaWebhook.ts — the shape every webhook handler has

## Change
A route `POST /stripe/webhook` registered in `convex/http.ts`, handled by `stripeWebhook.ts`:

```ts
export const stripeWebhook: HttpAction // (req) => Response
```

The handler verifies the signature with the shared `verifyStripeSig` helper before it reads the body. An unsigned or badly signed request gets 400 and writes nothing. A signed `checkout.session.completed` maps `metadata.orderId` to an `orders` row and sets its status from `draft` to `paid`. The same event a second time finds the row already `paid` and returns 200 with no write. An event with no matching `orderId` returns 200 and writes nothing. Every other event type returns 200 and writes nothing.

- Stays as it is: `convex/schema.ts`, the `orders` enum already carries `paid`; every other route.

## Acceptance Criteria
- AC-1: When a signature-valid `checkout.session.completed` arrives for a draft order, the handler shall set that order to `paid`
- AC-2: When that same event is replayed, the handler shall leave the order `paid` and fire no duplicate side effect
- AC-3: When an event arrives with no matching `orderId`, the handler shall return 200 and write nothing
- AC-4: When a request carries no valid signature, the handler shall return 400 and write nothing
```
