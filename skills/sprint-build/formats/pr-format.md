# Preamble

This file is the format of one pull request body. A ticket PR lands one ticket: one commit, one layer of a stack. A plan PR lands the records of a sprint plan, and the spec sweep of the sprint before it. The body is for the reviewer agents and for the person who decides to merge.

## File

- **Location:** The PR on GitHub. One body per PR. Nothing lives in a dotdir.
- **Title:** A ticket PR takes the subject line of its one commit, byte for byte, per `commit-format`. A plan PR takes `docs(plan): sprint-v{N}`.
- **Branch:** A ticket PR is `ticket/<N>-<slug>`. A plan PR is `docs/sprint-v{N}`.
- **Base:** A ticket PR's base is the PR of its blocker in the stack, or `main`. A plan PR's base is `main`. The position in a stack is a fact of GitHub's stack map, never of a body.
- **Labels:** A ticket PR carries `needs-review` when it opens and `needs-refine` after its review, never both. A plan PR carries no label.
- **Shape:** The body is flat `## ` sections, in template order, all present, no prose before the first. A parser uses `## ` to find each section.

## Template

```markdown
## Summary
<one or two sentences: why this change exists>

<the smallest view that shows the change, in a fence>

## Lands
Closes #N | <ID>[, <ID>]

## Evidence
<before and after>

## Merge Danger
- **Door:** one-way | two-way
- **Blast radius:** <one word>[ — <what can break>]
```

One entry per slot:

| Slot | Rule |
|---|---|
| **Summary** | Mandatory. First, one or two sentences: why the change exists, from the ticket or the records, never from the diff. Then the smallest view that shows the change: a diff sketch, a call tree, a file tree or a Mermaid diagram. One view, two at most. |
| **Lands** | Mandatory, single line. A ticket PR: `Closes #N`, the ticket its commit lands. A plan PR: the IDs of the new and changed records, `US-###` then `ADR-###`, the sprint plan's path, and `.spec/spec.md` when the sweep changed it. Never the text of a ticket or a record. |
| **Evidence** | Mandatory. A before and an after. A ticket PR: per criterion, the check from the ticket's `test:` line failing at the base and passing at the tip, then the build-order's `check` commands passing at the tip, as real output trimmed to the lines that matter. A plan PR: `None — records only.` |
| **Door** | Mandatory. `two-way` when a revert walks the merge back. `one-way` when it does not: a schema migration, deleted data, a removed route or field that a client reads, a message sent. |
| **Blast radius** | Mandatory. One word for the widest scope the change can affect: `none`, `module`, `subsystem`, `data`, `clients`. Then, when it is not `none`, what can break. |

## Mutability

- **Before the merge, the body is edited.** A fix after review changes the Evidence and, when needed, the Summary and Merge Danger. A ticket PR's title follows its commit's subject.
- **After the merge, the body is frozen.**

## Ownership

GitHub is the source of truth for what a PR proposed. The commit graph is the source of truth for what landed. Artifacts reference a ticket by `#N`, never a PR: a ticket PR is reached through its ticket. Nobody quotes a body. The draft state, the stack and the labels are GitHub facts that the skills set, never body slots.

## Example

Title: `feat(billing): mark an order paid on checkout.session.completed`

````markdown
## Summary
A paid checkout left its order in `draft`, because nothing listened for Stripe's completion event. Stripe now tells us, and the order moves to `paid` once, however often the event arrives.

```diff
 http.ts
   POST /luma/webhook   → lumaWebhook
+  POST /stripe/webhook → stripeWebhook
+    verifyStripeSig
+    checkout.session.completed → orders: draft → paid
```

## Lands
Closes #84

## Evidence
`pnpm vitest run convex/stripeWebhook.test.ts`

Before:
```
✗ AC-1 sets a draft order to paid
✗ AC-2 leaves a replayed event without a second write
✗ AC-3 writes nothing for an unmatched orderId
✗ AC-4 rejects an unsigned request with 400
```

After:
```
✓ AC-1 sets a draft order to paid
✓ AC-2 leaves a replayed event without a second write
✓ AC-3 writes nothing for an unmatched orderId
✓ AC-4 rejects an unsigned request with 400
```

`pnpm typecheck`, `pnpm lint`, `pnpm test`: pass.

## Merge Danger
- **Door:** two-way
- **Blast radius:** data — a wrong `orderId` mapping marks the wrong order paid
````
