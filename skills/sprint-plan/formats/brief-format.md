# Brief Format Reference

The Brief is the durable product charter — the slowest-changing artifact in the system. One file (`.brief/brief.md`). It states what this product is, who it serves, and the principles that guide it. `/sprint-plan` authors and reads it; the Sprint Plan, `/sprint-review`, and `/sprint-refine` **reference** it — they never reproduce its contents.

> [!IMPORTANT]
> The Brief is the one **living** exception in this system. Edit it **only on a strategic shift** (a change to vision, target users, value prop, principles, north-star, quality goals, non-goals, or Definition-of-Done). It is **NOT versioned** — there is no `brief-vN.md`, no version frontmatter, no `previous:` chain. **Git holds the history.** The Brief is a charter, not a discovery-dump: keep it tight, durable, and current. Cycle-specific findings, exploration notes, and per-feature detail belong elsewhere.

## Frontmatter

The Brief carries **exactly one** frontmatter field — nothing else:

```yaml
---
last_reviewed: {YYYY-MM-DD}
---
```

No `version`, no `status`, no `date`, no `author`, no `previous`. The Brief is not on the versioned PRD lifecycle.

## Template

```markdown
---
last_reviewed: {YYYY-MM-DD}
---

# {Product Name} — Brief

## Vision
{One sentence. The durable "why this exists."}

## Problem
{The durable problem this product solves. Not a cycle's problem — the standing one.}

## Target users
{One line. Who this is for. NOT a persona dossier.}

## Value proposition
{What we give them that they can't easily get elsewhere.}

## Principles
{2-5 durable product tenets — the tie-breakers for how this product makes decisions. See "Principles" below.}

## North-star metric
{The metric DEFINITION — what it measures and why it's the one number that matters.}
{Guardrails: metrics that must not regress while chasing the north-star. NO period targets.}

## Quality goals
{3-5 quality attributes, expressed as guardrails (arc42 §1.2). NOT targets.}

## Non-goals
{The durable things this product will never do. Not cycle scope — standing exclusions.}

## Definition of Done
{The canonical, project-wide DoD. This is its single home. Everything else references it.}
```

## Section caps

| Section | Cap |
| --- | --- |
| Vision | 1 sentence |
| Problem | Durable problem only — not a cycle's problem |
| Target users | 1 line — no persona dossier |
| Value proposition | Tight statement of the edge we offer |
| Principles | 2-5 durable product tenets (see below) |
| North-star metric | DEFINITION + guardrails — **NO period targets** |
| Quality goals | 3-5 attributes, as **guardrails NOT targets** |
| Non-goals | Durable standing exclusions only |
| Definition of Done | The single canonical DoD home |
| `last_reviewed` | One date in frontmatter |

## Principles

The `## Principles` section holds **2-5 durable product tenets** — the opinionated tie-breakers that decide how *this product* resolves a recurring trade-off, so the same argument is not re-litigated every cycle. A principle is a standing stance, not a one-off choice: when two valid options compete, it states which way this product leans by default.

These are principles of **the product being built**, at product altitude — not principles of the development process or the tooling.

**Belongs here** — durable product trade-off stances, e.g.:
- *"Privacy over personalisation — no user data leaves the device."*
- *"Correctness over latency — never show stale financial data, even if it costs a round-trip."*
- *"Reversible over fast — every destructive action must be undoable."*

**Does NOT belong here:**
- *How the workflow or tooling runs* (model choice, parallel-vs-sequential execution, agent roles) — that is the dev process, not the product.
- *A specific technology decision* ("Convex over Supabase") — that is one dated choice with alternatives → an **ADR**.
- *A measurable bar* ("p95 < 200ms", "99.9% uptime") → a **Quality goal**.

Distinguish from neighbours: a **principle** informs many future decisions; an **ADR** records one decision and its rejected alternatives; a **quality goal** is a measurable attribute. If a tenet is really about how the team or tooling builds rather than what the product values, it does not belong in the Brief.

## When to edit

Edit the Brief **only on a strategic shift**. A typo or a clarified phrasing is fine inline, but a real change — vision, target users, value prop, a principle, the north-star, a quality goal, a non-goal, or the DoD — is a strategic event. When you make one:

1. Apply the edit in place (no new file — git holds the before/after).
2. Bump `last_reviewed` to today.
3. **Pair it with a product ADR.** The Brief records **WHAT** shifted; the ADR records **WHY**. A strategic Brief change with no accompanying ADR is incomplete.

## Definition-of-Done ownership

The Brief is the **single canonical home** for the Definition of Done. Every other artifact that needs the DoD — the Sprint Plan's close-out gate, `/sprint-refine`'s DoD gate, `/sprint-review`'s quality check — **references** `.brief/brief.md#definition-of-done`. They do not restate it. There is exactly one DoD, and it lives here.

## What belongs elsewhere

The Brief is durable charter only. Faster-changing or more specific content lives in its proper artifact:

| Content | Lives in |
| --- | --- |
| Current set of wants ("As a X, I want Y, so that Z") | Stories (`.stories/STORIES.md`) |
| The WHY behind a decision (incl. a strategic Brief shift) | ADR (`.adr/ADR.md`) |
| What the system currently IS (architecture, data model, API) | Spec (`.spec/spec.md`) |
| This cycle's goal, scope, and timebox | Sprint Plan (`.sprint/sprint-vN.md`) |

## Greenfield

On a greenfield project there is no founding document to inherit from. The **first cycle** of `/sprint-plan` writes both the Brief (`.brief/brief.md`) and the first Sprint Plan (`.sprint/sprint-v1.md`) together. Every later cycle reads the existing Brief and only touches it on a strategic shift.

---

**Vocabulary:** `/sprint-plan`, `/sprint-review`, `.sprint/`, `.brief/`, `.stories/`, `.adr/`, `.spec/`
