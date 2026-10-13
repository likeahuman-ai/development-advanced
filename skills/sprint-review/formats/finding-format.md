# Preamble

This file is the format of the `### Code Review` comment: the published review of one ticket PR. `/sprint-review` posts it; `/sprint-refine` reads it as the PR's work order. Fix findings are what Refine fixes. Note findings are for the person who reads the PR.

## Gate

A finding goes under **Fix** when it is one of these:

1. **A requirement that is missing, partial or wrong** — the change does not do what the ticket asks.
2. **Behaviour the ticket did not ask for** — scope creep, or a breach of the ticket's `Stays as it is:` line.
3. **A test that does not prove its criterion** — it passes without the change, or its expected value is computed the way the code computes it.
4. **A breach of a documented standard** — a rule in the repository's `CLAUDE.md`, its coding standards, a decision on the ticket's `Related to` line, or `.spec/spec.md`.
5. **A real defect** — a bug, a swallowed error or a security flaw, with evidence in the code.

Every other finding is a **Note**: a smell, a naming question, a judgement call. A finding that the build-order's check commands already catch is dropped: those commands pass on every PR.

## File

- **Location:** A comment on the PR, one per review round. A new round posts a new comment; an earlier one is never edited.
- **Shape:** The first line is `### Code Review`, verbatim: the parse anchor. Then the scope line and the count line. Then `#### Fix` and `#### Note`, in that order, each present, each holding finding blocks or the line `None.`
- **Numbering:** Each finding is `F-n`. The sequence is per PR and append-only across rounds: a later round continues from the highest number any earlier `### Code Review` comment on the PR used. A number is never reused. Outside the comment, a finding is `#<PR> F-n`.

## Template

````markdown
### Code Review

<one line: what was reviewed>
Fix: <n> · Note: <n>

#### Fix

**F-<n>: <title>**
Axis: spec | standards | quality | security · Found by: <agent>
<the claim: what is wrong, where, and why, one concern>
Evidence: <the facts in the code; may cite code outside the diff>
[Expected: <the correct behaviour, one line>]
Edits: `<path>`[, `<path>`]
[permalink](https://github.com/<owner>/<repo>/blob/<sha>/<path>#L<start>-L<end>)

#### Note

**F-<n>: <title>**
Axis: … · Found by: …
…
````

One entry per slot:

| Slot | Rule |
|---|---|
| **scope line** | Mandatory. One line naming the reviewed change and its ticket, `#N`. |
| **count line** | Mandatory. `Fix: <n> · Note: <n>`, the number of blocks in each section. `Fix: 0` means the PR is reviewed clean. |
| **title** | Mandatory. `**F-n: <title>**`, a few words. |
| **Axis** | Mandatory. The review axis that found it: `spec`, `standards`, `quality` or `security`. Findings from different axes are never merged into one block. |
| **Found by** | Mandatory. The `name:` of the reviewer agent. |
| **claim** | Mandatory. One concern: what is wrong, where, and why, in a sentence or two. Two concerns are two findings. |
| **Evidence** | Mandatory. The facts the claim rests on: the code, the ticket line, the rule. It can cite callers, types and tests outside the diff. |
| **Expected** | Present when the claim is about behaviour: the correct behaviour in one line, Given/When/Then, or EARS when it depends on prior state. The fix's regression test encodes it. |
| **Edits** | Mandatory under Fix. Every path the fix edits, and only those, so the fix agent knows where to work and a reader knows what it may touch. |
| **permalink** | Mandatory. Pinned to the PR head's full 40-character SHA, with a line range `#L<start>-L<end>` even for one line. The owner and repository come from the repository's remote. |

## Mutability

- **A posted comment is frozen.** A new round posts a new comment, and its numbers continue.

## Ownership

The PR's latest `### Code Review` comment is its work order. Refine fixes every Fix finding and leaves every Note. Artifacts reference a finding by `#<PR> F-n`. Nobody quotes a finding.

## Example

````markdown
### Code Review

Reviewed #212 — the Stripe webhook handler for #84.
Fix: 2 · Note: 1

#### Fix

**F-1: A replayed event writes the order a second time**
Axis: spec · Found by: spec-reviewer
AC-2 asks for no write on a replay, but the handler patches the row before it checks the status.
Evidence: `convex/stripeWebhook.ts:41` calls `ctx.db.patch` before the `status === "paid"` check at `:44`.
Expected: When the same `checkout.session.completed` arrives twice, the handler shall write the order once.
Edits: `convex/stripeWebhook.ts`
[permalink](https://github.com/acme/shop/blob/3f2a9c1d4b8e6705a1c2d3e4f5061728394a5b6c/convex/stripeWebhook.ts#L41-L44)

**F-2: The signature is read after the body is parsed**
Axis: security · Found by: security-reviewer
The handler parses the JSON body before `verifyStripeSig` runs, so a forged body reaches the parser.
Evidence: `convex/stripeWebhook.ts:22` calls `req.json()`; `verifyStripeSig` runs at `:27`. ADR-009 requires the check first.
Expected: Given a request with no valid signature, When it arrives, Then the handler returns 400 before it reads the body.
Edits: `convex/stripeWebhook.ts`
[permalink](https://github.com/acme/shop/blob/3f2a9c1d4b8e6705a1c2d3e4f5061728394a5b6c/convex/stripeWebhook.ts#L22-L27)

#### Note

**F-3: The event-type switch repeats the router's switch**
Axis: standards · Found by: standards-reviewer
Possible Repeated Switches: the handler and `convex/http.ts:52` both switch on the Stripe event type.
Evidence: `convex/stripeWebhook.ts:30-38` and `convex/http.ts:52-60`.
[permalink](https://github.com/acme/shop/blob/3f2a9c1d4b8e6705a1c2d3e4f5061728394a5b6c/convex/stripeWebhook.ts#L30-L38)
````
