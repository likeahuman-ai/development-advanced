# ADR Format Reference

Architecture Decision Records capture the "why" behind architectural choices. One file (`.adr/ADR.md`), append-only, numbered sequentially.

## Entry Template

```markdown
## ADR-{NNN}: {verb-phrase title}

> In the context of {situation}, facing {concern}, we decided {decision}
> to achieve {goal}, accepting {tradeoff}.

**Date:** {YYYY-MM-DD}
<!-- Add a **Status:** line ONLY for `proposed`, `superseded`, or `deprecated`. Accepted is the default and is left implicit. -->

**Context:** {1-2 sentences — what forced the decision}

**Decision:** {what was chosen}

**Alternatives rejected:**
- {Alternative A} — {one-line reason}
- {Alternative B} — {one-line reason}

**Locks in:** {the forward constraint this commits us to}

**Makes harder:** {the cost — what this rules out or complicates}

**Scope:** {coarse subsystem boundary this decision governs}

**Revisit when:** {trigger condition}
```

## Decision Threshold

Create an ADR when the decision:
- Crosses a module or service boundary
- Introduces a new dependency (library, service, API)
- Constrains future options (locks in a pattern or approach)
- Would be argued about again in 6 months

## NOT an ADR

Do not create ADRs for:
- Naming choices (variable names, file names)
- Reversible configuration (env vars, feature flags)
- Implementation details within a single module
- Choices with no meaningful alternatives (only one option exists)
- Standard framework patterns (using Next.js app router when the project is Next.js)

## Anti-Bloat Guardrail

An ADR entry is a **decision plus a one-line rationale** — nothing more. If an entry starts needing cost tables, benchmark measurements, research appendices, or option-synthesis maps to make its case, it is no longer an ADR:

- A multi-option tradeoff study with measurements is a **design doc** — that work belongs in the per-cycle `code-architect` design (no design-doc file; it lives in the cycle) or, if durable, as a `.spec/spec.md` section.
- Background research and synthesis maps belong in the Spec's "Crosscutting Concepts & Patterns" section, not the ADR.

Keep the ADR to the Y-statement, the trigger, the alternatives (one line each), the consequences, and the revisit condition. If you can't say it that compactly, the decision isn't ripe — or it's a Spec change.

## File Conventions

- Location: `.adr/ADR.md` (single file)
- Numbering: sequential, zero-padded three digits (ADR-001, ADR-002, ...)
- Append-only: new entries go at the bottom
- To reverse a decision: add a NEW ADR with `**Supersedes:** ADR-{NNN}`
- Never edit or delete existing entries

## Section Guidance

**Y-statement** (the blockquote): MANDATORY on every entry. One sentence that gives the decision at a glance. A reader scanning the file should understand each decision from the blockquote alone. No ADR ships without it.

**Status:** Omit it for the common case — accepted is the default and stays implicit. Add an explicit `**Status:**` line ONLY when the entry is `proposed` (not yet ratified), `superseded` (a later ADR replaced it), or `deprecated` (no longer recommended but not yet replaced).

**Context:** What forced this decision NOW? Not background — the trigger.

**Alternatives rejected:** Minimum 2 real alternatives. "Do nothing" counts if it was genuinely considered. Each needs a specific rejection reason, not "didn't fit."

**Scope:** Name the COARSE subsystem boundary this decision governs — for human orientation only, not a file contract. Use a subsystem name, not file paths or globs. Example: `the backend persistence layer`. The machine-enforced file contract is the ticket Write-set; the implementer never receives the ADR, so Scope does not constrain which files get touched — it just tells a reader roughly where the decision lives.

**Revisit when:** A testable condition. "When we need more flexibility" is bad. "When monthly API calls exceed 10K or a second consumer needs the same data" is good. This line also hosts deferred architectural-debt triggers — record the condition under which a known shortcut must be paid down here.

## Example: Good ADR at the threshold

```markdown
## ADR-003: Convex over Supabase for backend

> In the context of choosing a backend for a real-time collaboration app,
> facing the need for live-updating presence data, we decided Convex
> to achieve built-in reactivity without WebSocket boilerplate, accepting
> vendor lock-in and a smaller ecosystem.

**Date:** 2026-05-05

**Context:** The platform needs real-time presence, live progress
tracking, and instant updates when shared state changes. Both Convex and
Supabase were evaluated.

**Decision:** Convex as the sole backend (database + functions + real-time).

**Alternatives rejected:**
- Supabase — requires manual WebSocket setup for real-time, Postgres requires schema migrations, more operational overhead for a small team
- Firebase — vendor lock-in without the developer experience benefits, weaker TypeScript support

**Locks in:** All backend logic in Convex functions; no SQL access; real-time only within Convex's model.

**Makes harder:** Migrating to another DB (rewrite every query); anything needing joins across data outside Convex.

**Scope:** the backend persistence and real-time data layer

**Revisit when:** Team exceeds 5 engineers (Convex's single-writer model may become a bottleneck) or we need joins across data that lives outside Convex.
```

(Note: the example carries no `**Status:**` line — accepted is the default. The blockquote Y-statement is present and mandatory.)

## Example: Too small for an ADR (belongs in Sprint Plan/ticket)

"We'll put the Button component in `src/components/ui/button.tsx`" — this is a file placement decision with no meaningful alternative and zero future consequence. It stays in the ticket body.
