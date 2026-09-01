# ADR Format

Shape of `.adr/ADR.md` — the append-only architecture decision log. Each record is **standing law**: written once at 1.2.4, consumed context-less by every later phase (*trust the artifact*). Written for a model to read back, not a human to sign off (*artifacts are for AI, not humans*) — no approval boxes, no restated context, no record nothing points at.

## File conventions

- **Location:** `.adr/ADR.md` — one file, every record, appended in creation order. (The dotdir is `.adr/`; the file is `.adr/ADR.md`.)
- **File shape:** **no H1, no preamble** — the file is the records and nothing else. The first meaningful line is the first record's `## ADR-001` heading; every record is an `## ADR-###` block (`## ` is the record delimiter a parser keys on). No title, no intro, no separators between records.
- **Numbering:** `ADR-###` — sequential, zero-padded three digits (`ADR-001`, `ADR-002`, …). No gaps, never reused, never out of order.
- **Append-only:** a record's body is never edited or deleted. Reverse a decision by appending a **new** record carrying `Supersedes: ADR-###`. The only sanctioned touch on an existing record is adding one `Status:` line — an annotation, not a rewrite.
- **Pointed at, never copied:** other artifacts and commit trailers reference a record by `ADR-###` only; the body lives once, here. The point-in-time text is recoverable from history (`git show <commit>:.adr/ADR.md`) — so IDs must stay stable.
- **The commit binding lives outside this file — by design.** Which commit introduced or last touched a record is **not** a field here; that binding is `commit-format`'s `ADR: ADR-###` trailer on the `docs(plan)` commit — a trailer that points at the record by ID, never a copy — recovered from git, never restated in the record. A reader of a checked-out `ADR.md` sees only IDs + content; the commit that pinned each record is found by `git log --grep` on the trailer, not from any in-record marker. This format owns the record's **shape**; `commit-format` owns the **commit ↔ record** link.

## The bar — what earns a record

Three judgment tests, applied **as guides, never a hard all-three gate**: the decision is **hard to reverse** · **surprising without context** · **a real trade-off** was made. A decision is captured on judgment guided by these tests — a governing decision that fails one test may still earn a record; the producing skill (1.2.4) owns when the bar is met, and when in doubt the reasoning stays in the plan.

Not records: naming choices · reversible config · single-module implementation detail · a choice with no real alternative · standard framework patterns. **Scarcity is the design** — every record is load-bearing law, so each reads terse: no cost tables, no measurements, no alternatives studies, no design-doc prose. Bloat is the signal a record didn't clear the bar.

## Template

```markdown
## ADR-### — <decision title>

> In the context of <context>, facing <concern>, we decided <decision> to achieve <goal>, accepting <trade-off>.

- **Date:** YYYY-MM-DD
- **Context:** <the trigger that forced this decision now>
- **Decision:** <what was chosen, declarative>
- **Alternatives rejected:**
  - <alternative actually weighed> — <one-line reason>
  - … one line per alternative genuinely considered
- **Locks in:** <forward constraints this creates>
- **Makes harder:** <costs / options now ruled out>
- **Scope:** <coarse subsystem boundary>[ — cross-cutting if it binds every consumer]
- **Revisit when:** <testable trigger condition>
```

## Field rules

| Field | Rule |
|---|---|
| **Y-statement** | Mandatory. One sentence, blockquote, directly under the heading: *context · concern · decision · goal · trade-off*. The opening anchor — and the **only** part lifted **verbatim** into downstream prompts (architect 2.2.1, implementer 3.2.1) as a non-negotiable constraint, so it must stand alone: decision, goal, and trade-off legible without the body. |
| **Date** | Mandatory. `YYYY-MM-DD` — the day the decision was made (capture date at 1.2.4), **not** the commit date or any later edit. A point-in-time fact: it stays frozen at the value written, never re-dated when the record is committed, rebased, or annotated. Git owns the commit timestamp; this field owns the decision moment. |
| **Context** | Mandatory. The **trigger** — what forced this decision now — not background. When an artifact owns the trigger fact, **cite it** (`US-###`, `.spec/spec.md#anchor`, `#N`, `file:line`) — never restate a fact that lives elsewhere. When no artifact owns it — an inherent domain fact (e.g. "clients disconnect without an explicit logout"), common on a greenfield sprint with no spec anchors or code yet — **state it as bare prose**: one line, no citation. The cite-or-prose call separates a legitimate ownerless trigger from a banned restatement: if a fact has an owner, the citation is mandatory; prose is for facts that have none. |
| **Decision** | Mandatory. What was chosen, stated declaratively. |
| **Alternatives rejected** | Mandatory. **Only the alternatives that were genuinely weighed**, each a one-line reason — record what was actually considered, never manufacture options to hit a count (an invented alternative reads downstream as a real decision). A real trade-off (the third bar test) almost always leaves ≥2 weighed; one that leaves none didn't clear the bar. One line means one line — no comparison study. |
| **Locks in** | Mandatory. The forward constraints this creates for later work. |
| **Makes harder** | Mandatory. The cost — options now ruled out or made expensive. |
| **Scope** | Mandatory. **Coarse** subsystem boundary (`payments`, `auth`, `build pipeline`) — human orientation only, **never a file-path contract**. Implementers receive the Y-statement + decision, never this field. Mark **breadth** with one of two forms, so a consumer can tell a subsystem-local record from cross-cutting law: `payments` (local — governs that subsystem) · `payments — cross-cutting` (binds **every** consumer of that subsystem, not just code inside it). Cross-cutting is the signal a consumer reads in full as standing law even when no plan names it; local records are safe to read selectively when the plan doesn't call them out. |
| **Revisit when** | Mandatory. A **testable** trigger ("provider ships X", "load exceeds Y") — never "periodically". |
| **Status** | Optional, single line — the **only** sanctioned edit to a written record. Closed value set: `proposed` · `superseded by ADR-###` · `deprecated`. **Absence is not missing data — it definitionally means `accepted`** (the default, never written explicitly), so every record's lifecycle is decidable whether or not the line is present. No value outside the set is valid. |
| **Supersedes** | Optional. `Supersedes: ADR-###` — first list line, only on a record that reverses an earlier one. |

## Example

```markdown
## ADR-004 — Idempotency keys on all payment mutations

> In the context of checkout retrying failed network calls, facing the risk of double-charging on retry, we decided to require a client-generated idempotency key on every payment mutation to achieve exactly-once charge semantics, accepting an extra key table and key-management burden in every payment client.

- **Date:** 2026-06-02
- **Context:** US-014 retries checkout on timeout; load testing surfaced duplicate charges where the first request had already succeeded server-side (`checkout.ts:88`).
- **Decision:** Every payment mutation takes a required `idempotencyKey`; the server stores key → result and replays the stored result on a repeated key.
- **Alternatives rejected:**
  - Dedup by (user, amount, 5-min window) — heuristic; swallows legitimate rapid repeat purchases.
  - Provider-side idempotency only — covers the charge, not our own order-row writes; partial duplicates remain.
- **Locks in:** every payment client (web, mobile, backfill scripts) generates and persists keys across retries.
- **Makes harder:** fire-and-forget payment calls; ad-hoc manual testing against payment endpoints.
- **Scope:** payments
- **Revisit when:** the provider exposes end-to-end idempotency covering our order writes, or payment mutations move fully provider-side.
```

## Supersession

The append-only reversal (File conventions) worked end to end — a **new** record plus the one-line `Status:` annotation on the old:

New record:

```markdown
## ADR-009 — Provider-managed idempotency

> In the context of …

- **Supersedes:** ADR-004
- **Date:** 2026-09-15
…
```

On `ADR-004`, append one line:

```markdown
- **Status:** superseded by ADR-009
```

The two fields are reciprocal: `Supersedes:` on the new record points back, `Status:` on the old points forward — so the governing-vs-superseded state of any record is decidable from the records alone, with no external index.
