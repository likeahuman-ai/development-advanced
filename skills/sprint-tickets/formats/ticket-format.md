# ticket-format

Shape of one GitHub issue body — the AI-ready work order for one ticket. Written at 2.4.4 (`gh issue create --body-file`); read in full at Build 3.1.4 (`gh issue view`) as the raw material for an implementer prompt (3.2.1), and threaded by `#N` for wave ordering.

**Form principle — one ticket = one independently verifiable change.** A clean ticket maps 1:1 to one atomic commit at Build; if the backstop has to split it, the ticket carried two concerns — a decomposition miss. Traceability is mandatory; off-limits facts live in the artifacts this body points at.

**Binding vs advisory — the split is structural.** `## Requirements`, `## Acceptance Criteria`, and `## Constraints` **bind** — the implementer executes them as written. `## Context` **localizes, never instructs** — its anchors say where to look, not what to do; the implementer diagnoses against the code it finds there, and a Context gloss that turns out wrong yields to the tree. An approach suggestion, when one is worth carrying, is **advisory**: it rides in `## Context` marked `suggestion:` — the implementer may override it after its own diagnosis, and overriding it is not a deviation. (A misleading hint followed over a correct diagnosis is the documented failure mode this split exists to prevent.) The one exception inside Context: the `Expected behaviour:` line is a compressed restatement of the AC's oracle — **binding like the AC it mirrors**, never an overridable gloss.

## Reference, never reproduce

This body **points at** facts owned elsewhere — it never copies them:

- `US-###` → the want lives in `.stories`; cite the ID, not the story text.
- `ADR-###` → the decision + its rationale live in `.adr`; cite the ID, not the Y-statement.
- `.spec/spec.md#anchor` → the documented behaviour lives in `.spec`; cite the anchor, not the prose.
- `#N` (`Part of` · `Blocked by` · `Blocks`) → the parent/peer ticket owns its own body.
- The DoD lives in `.brief` — never restate it here (AC is *this ticket's* slice of done, not the project bar).
- Business justification (why this sprint) lives in the `.sprint` plan — never restated.
- The Verify command lives in the build-order `## Verify` — never restated in AC.

No sign-off boxes, no approver/timestamp fields, no requirement IDs nothing references, no `CLAUDE.md` echoes. *Artifacts are for AI, not humans.*

**The inline `<!-- … -->` comments below are scaffolding, not content** — author guidance the producer reads and deletes; they never ship in an issue body, and a consumer never parses them. So any semantics a section depends on are stated **in the section's own emitted lines**, never carried by a gloss: a bare `## Complexity` marker means resource-cost by the rule in §4 (the consumer recovers the meaning from the section, not from a comment that won't be there). The `## Example` shows a real body with the comments gone.

## Sections — in this order

Flat sections, no nested subsections, no prose preamble. Each serves one downstream role.

1. `## Objective` — one sentence: this ticket's scope + intent (from the architect's design unit). Read at 3.2.1 as the task the implementer is handed.
2. `## Traceability` — bullet list: `Story:` `US-###` · `Sprint:` `v{N}` · `Epic:` `#N (epic:<name>)` **or** `—` (a flat breakdown has no epic issue or label) · `ADR:` `ADR-###` (governing records, space-separated) **or** `—`. `Story:`/`ADR:` forward to commit trailers (3.2.4 — `commit-format`'s trailer table is closed; `Sprint:`/`Epic:` never become trailers); `Sprint:`/`Epic:` serve coverage derivation (5.3.2) only. The `epic:<name>` label is a **case-sensitive forge identifier** — copy it **verbatim**, byte-for-byte (casing, hyphens, separators), never normalized; the consumer filters/groups epics on the exact string (`epic:Stripe-integration` ≠ `epic:stripe-integration`).
3. `## Context` — anchors that localize the implementer's work; scoped to the touched modules (from 2.1.1). One line per touched location, each with a fixed prefix so the consumer reads granularity off the line, not by guessing:
   - **existing file** → `file:line` (`auth.ts:42`) — the seam the change lands at.
   - **file this ticket creates** → `new: <path>` (no line — there's no line yet), e.g. `new: src/payments/stripe/adapter.ts`. The `new:` prefix is the format's rule for the created-vs-existing tension: a created path can't carry a `file:line` anchor, so it carries directory-and-name granularity and is marked as such — never a bare path the consumer can't classify.
   - `.spec/spec.md#anchor` (if present, else omit the line) · expected behaviour in one or two lines.
   - **advisory approach suggestion** (optional) → `suggestion: <one line>` — advisory by the binding-vs-advisory split above; the implementer may override it after its own diagnosis.
4. `## Complexity` — one marker on its own line, `S` · `M` · `L`, and nothing else. The marker **means resource-cost** (effort + blast radius), **never** time — that semantics is fixed here, so a bare `M` is unambiguous to the consumer with no gloss present. Drives implementer dispatch (3.2.1).
5. `## Requirements` — concrete, verifiable checklist (`- [ ]`). Scope items; injected into the implementer prompt (3.2.1).
6. `## Acceptance Criteria` — Given/When/Then **unless** the success condition depends on prior state or an accumulated count (then EARS). Operational test: if a criterion can't be written without naming what happened *before* — a retry tally, a prior flip, a backoff window, an idempotency replay — it's stateful → EARS (`When <trigger>, the <system> shall <response>`); otherwise GWT. One ticket renders all its criteria in one notation. **Every criterion carries its number:** the bullet opens `AC-n:` — sequential from `AC-1:` in emission order, before the GWT/EARS text. The prefix is identity, not structure — notation and density unchanged; the ID is stable — a later edit never renumbers existing criteria, and the sequence is **append-only**: a criterion added later takes the highest number this body has ever carried plus one; a struck criterion's number is never reused and its gap stays (`AC-1 · AC-3 · AC-4` is a conformant body). **Citation grammar:** outside this issue a consumer cites the compound coordinate `#<issue> AC-n` (issue number + criterion — e.g. `#84 AC-3`); bare `AC-n` is local to this body only, meaningless without its issue. Verifiable success; self-checked at 3.2.1, re-checked at 3.2.5 spec-review. Never restate the build-order Verify.
7. `## Constraints` — off-limits files + must-use patterns (from `.adr` governance). Injected into the implementer prompt as guardrails (3.2.1).
8. `## Dependencies` — one `Blocked by:` line (the upstream tickets this one consumes — validated against wave ordering at 3.1.4), a **single space-separated list** of `#N` IDs (`Blocked by: #104 #16 #17`); `—` when none. One delimiter only — never comma- or newline-separated — so the consumer splits on whitespace. There is **no `Blocks:` line**: creation runs in dependency order (upstream first, 2.4.4), so downstream numbers cannot exist when an upstream body is written — and `Blocked by:` already encodes every edge in the direction the waves consume.

**Mandatory:** all sections except where marked `or —` — the `Epic:` line, the `ADR:` line, and the `.spec` anchor are the only optional facts. No `## Files` footprint section exists — no step produces per-file attribution (worktrees removed the need); localization rides the `## Context` anchors. Density target: ~150–300 words of prose + lists per ticket.

**Output-shaped tickets carry a sample.** When a ticket's job is to *parse or build against the output of a tool the implementer cannot run* — it provisions only the build and never runs git or project CLIs (implementer-prompt) — embed a **representative sample of that output verbatim** in `## Context` (a fenced block; or `## Requirements` if the shape *is* the spec), captured at Tickets time **by the session** (the explorers and architects are read-only — Read/Glob/Grep — and cannot run the tool; the session runs it once and embeds the live output, or embeds a documented sample marked `(expected)` only when the session **cannot execute** the tool — unavailable in the environment, credential-gated, or with no runnable target yet; never mere inconvenience). The implementer builds against the sample, never a live run — so the absolute "never run git" rule stays intact *and* the contract stays observable. E.g. a ticket to parse `gh pr list --json` output includes a real snippet of that JSON.

## Template

```markdown
## Objective
<one sentence — what this change is>

## Traceability
- Story: US-###
- Sprint: v{N}
- Epic: #N (epic:<name>)
- ADR: ADR-### ADR-###   <!-- governing records, or — -->

## Context
- <file:line> — <why it's relevant>           <!-- existing file: file:line -->
- new: <path>                                  <!-- file this ticket creates: no line -->
- Spec: .spec/spec.md#anchor                   <!-- omit line if no .spec -->
- Expected behaviour: <one line>

## Complexity
M

## Requirements
- [ ] <concrete, verifiable item>
- [ ] <concrete, verifiable item>

## Acceptance Criteria
<!-- GWT unless success depends on prior state/count → then EARS; one notation per ticket; every bullet numbered AC-n: sequential from AC-1 -->
- AC-1: Given <state>, When <action>, Then <observable outcome>
- AC-2: Given <state>, When <action>, Then <observable outcome>
<!-- EARS form: AC-n: When <trigger>, the <system> shall <response> -->

## Constraints
- Off-limits: <files/dirs not to touch>
- Must use: <pattern from ADR-###>

## Dependencies
- Blocked by: #N #N   <!-- space-separated #N IDs; — when none; no Blocks: line exists -->
```

## Example

The example body below shows the comments gone, per the scaffolding rule.

```markdown
## Objective
Add a Stripe webhook handler that marks an order paid on `checkout.session.completed`.

## Traceability
- Story: US-014
- Sprint: v7
- Epic: #102 (epic:Stripe-integration)
- ADR: ADR-009

## Context
- convex/http.ts:48 — HTTP router; webhook routes register here
- convex/orders.ts:96 — `orders` table + status enum
- new: convex/stripeWebhook.ts — the handler module this ticket adds
- Spec: .spec/spec.md#payments-webhooks
- Expected behaviour: a verified Stripe event flips the matching order draft→paid; unmatched events 200 and no-op

## Complexity
M

## Requirements
- [ ] Verify the Stripe signature before reading the body; reject unsigned with 400
- [ ] Map `checkout.session.metadata.orderId` to an `orders` row
- [ ] Flip that order draft→paid idempotently (replayed event is a no-op)
- [ ] Return 200 for any unmatched event id

## Acceptance Criteria
- AC-1: When a signature-valid `checkout.session.completed` arrives for a draft order, the handler shall set that order to `paid`
- AC-2: When that same event is replayed, the handler shall leave the order `paid` and fire no duplicate side effect
- AC-3: When an event arrives with no matching `orderId`, the handler shall return 200 and write nothing

## Constraints
- Off-limits: convex/schema.ts — the `orders` enum already carries `paid` (ADR-009)
- Must use: the shared `verifyStripeSig` helper per ADR-009; no inline secret handling

## Dependencies
- Blocked by: #104
```
