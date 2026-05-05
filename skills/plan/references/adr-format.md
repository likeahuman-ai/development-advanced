# ADR Format Reference

Architecture Decision Records capture the "why" behind architectural choices. One file (`.adr/ADR.md`), append-only, numbered sequentially.

## Entry Template

```markdown
## ADR-{NNN}: {verb-phrase title}

> In the context of {situation}, facing {concern}, we decided {decision}
> to achieve {goal}, accepting {tradeoff}.

**Status:** accepted
**Date:** {YYYY-MM-DD}

**Context:** {1-2 sentences — what forced the decision}

**Decision:** {what was chosen}

**Alternatives rejected:**
- {Alternative A} — {one-line reason}
- {Alternative B} — {one-line reason}

**Consequences:** {what this locks in or makes harder}

**Scope:** {file paths this decision governs}

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

## File Conventions

- Location: `.adr/ADR.md` (single file)
- Numbering: sequential, zero-padded three digits (ADR-001, ADR-002, ...)
- Append-only: new entries go at the bottom
- To reverse a decision: add a NEW ADR with `**Supersedes:** ADR-{NNN}`
- Never edit or delete existing entries

## Section Guidance

**Y-statement** (the blockquote): One sentence that gives the decision at a glance. A reader scanning the file should understand each decision from the blockquote alone.

**Context:** What forced this decision NOW? Not background — the trigger.

**Alternatives rejected:** Minimum 2 real alternatives. "Do nothing" counts if it was genuinely considered. Each needs a specific rejection reason, not "didn't fit."

**Scope:** List the file paths or patterns this decision governs. Downstream agents (code-architect, implementer) use this to know which files are constrained. Example: `src/api/**, convex/schema.ts, package.json (dependencies section)`

**Revisit when:** A testable condition. "When we need more flexibility" is bad. "When monthly API calls exceed 10K or a second consumer needs the same data" is good.

## Example: Good ADR at the threshold

```markdown
## ADR-003: Convex over Supabase for backend

> In the context of choosing a backend for a real-time workshop platform,
> facing the need for live-updating participant data, we decided Convex
> to achieve built-in reactivity without WebSocket boilerplate, accepting
> vendor lock-in and a smaller ecosystem.

**Status:** accepted
**Date:** 2026-05-05

**Context:** The platform needs real-time participant status, live progress
tracking, and instant updates when workshop state changes. Both Convex and
Supabase were evaluated.

**Decision:** Convex as the sole backend (database + functions + real-time).

**Alternatives rejected:**
- Supabase — requires manual WebSocket setup for real-time, Postgres requires schema migrations, more operational overhead for a small team
- Firebase — vendor lock-in without the developer experience benefits, weaker TypeScript support

**Consequences:** All backend logic in Convex functions. No SQL access. Migration path to another DB is expensive (rewrite all queries). Real-time is free but only within Convex's model.

**Scope:** `convex/**, src/lib/convex.ts, .env.local (CONVEX_URL)`

**Revisit when:** Team exceeds 5 engineers (Convex's single-writer model may become a bottleneck) or we need joins across data that lives outside Convex.
```

## Example: Too small for an ADR (belongs in PRD/ticket)

"We'll put the Button component in `src/components/ui/button.tsx`" — this is a file placement decision with no meaningful alternative and zero future consequence. It stays in the ticket body.
