# Brief format

The shape of `.brief/brief.md` — the product charter and the **single home** of the Definition of Done. One file per project; the slowest-changing artifact (1.2.1 touches it only on greenfield, brownfield first contact, or a discovery-surfaced project-level shift). It holds only what is **durable across sprints**: if a line would change next sprint, it belongs in `.sprint`/`.stories`/`.adr`/`.spec`, not here.

## File

`.brief/brief.md`. Headings exactly as in the template, in this order, so anchors like `.brief/brief.md#definition-of-done` and `#quality-goals` resolve. Sections are all required.

## Template

```markdown
---
last_reviewed: {YYYY-MM-DD}
---

# {Product name}

## Vision
{the durable why this product exists}

## Problem
{the standing problem this product solves}

## Target users
{who this is for}

## Value proposition
{the edge over alternatives}

## Principles
- **{Stance}** — {how this product resolves a recurring trade-off}{; where a Quality goal enforces it, name that goal — never the bound}.
{2–5 items}

## North-star metric
**{Metric}** — {what it measures and why it's the one number that matters}. {definition only}

## Quality goals
- **{Guardrail name}** — {one bar, one measurable bound, threshold inline}.
{≥3 distinct bars; aim for 3–5, but never pack two bars to fit}

## Non-goals
- {What this product will never do}

## Definition of Done
A change is done when:
- {Per-PR criterion — judgeable against one PR as it lands.}

Outcome criteria (judged across the sprint, not one PR):
- {Outcome criterion — e.g. the sprint's intent still serves the north-star. Lives here when it can't be checked against a single PR, so the fact isn't dropped.}
```

## Field rules

| Section | Rule |
|---|---|
| Frontmatter | `last_reviewed: YYYY-MM-DD` only — set to today on every deliberate review, including a no-change one (distinguishes "checked, still true" from stale). No `version`, `status`, `author`, or `previous` — the Brief is **not** on a versioned lifecycle. |
| Vision | One sentence. The durable "why this exists." No elaboration. |
| Problem | The standing problem, in durable form. This sprint's problem lives in `.sprint/sprint-v{N}.md`. |
| Target users | One line. No persona dossier. |
| Value proposition | Tight: the edge — what they get here they can't easily get elsewhere. |
| Principles | 2–5 durable product tenets — standing stances on recurring trade-offs (privacy vs personalisation, correctness vs latency, reversibility vs speed). Product-level only: a technology choice is an `ADR-###`; a dev-process stance is not a product tenet. A captured `ADR-###` is checked against these (1.2.4) — flag, don't restate. When a principle's stance is also enforced by a Quality goal (e.g. a privacy stance and a privacy bound), the principle gives only the stance + its scope and **names** that goal as where the measurable edge lives — it never restates the bound (the Quality goal is its single home, exactly as the DoD never restates one). The stance lives here, the number in the goal — the canonical home is never ambiguous. |
| North-star metric | The **definition** only — what it measures and why it's the one number. Never a period target ("1M users by Q4" is banned shape) and never a guardrail list — a metric that must not regress while chasing it is a Quality goal (with its bound), not a line here. The definition feeds the 1.1.1 intent check and 1.2.4 ADR check. |
| Quality goals | The **single home** of every standing quality bar — quality attributes as measurable **guardrails** (arc42 §1.2), standing through sprints. **One bar = one named line, one bound** ("p95 < 200ms on the production dataset", "zero silent failures", "recovery ≤ 5 minutes") — never two distinct bounds on one line (the second loses its name and stops being judgeable). The bar's name and its number are never split apart, north-star guardrails included: a metric that must not regress while chasing the north-star is one of these distinct bars, listed here with its bound, never under North-star. **Aim for 3–5; the single-home mandate wins the tie** — when distinct standing bars (north-star guardrails among them) exceed 5, the count of *distinct* bars governs, never the cap: list every one as its own line, don't pack two to fit. (Above ~6, that is itself a signal the product carries too many standing bars — surface it, don't bury it.) These are read by the Tickets session at 2.0.2 (the working inputs), form part of the review standard reviewers judge against (4.2.2), and gate the DoD's no-regress criterion (5.2.1); write each so an agent can judge a change against it with no further context. |
| Non-goals | Standing exclusions — what this product will *never* do. Not sprint scope ("we won't build X this sprint" → `.sprint`). |
| Definition of Done | The canonical project-wide DoD, the **single home** of this fact, applied per-PR at acceptance (5.2.1); the plan's DoD-ref (1.2.6) points here. Two parts: **per-PR criteria** ("a change is done when…", judgeable against one PR — the per-PR DoD is the project DoD applied to what that PR delivers), and **outcome criteria** judged across the sprint (e.g. the intent still serves the north-star) — keep these in their own list so a real DoD fact that can't be checked against one PR is *recorded*, never silently dropped. The no-regress criterion points at Quality goals for its thresholds (their single home) — it never restates a bound. |

## What does not belong

- Sprint goal, scope, success metric → `.sprint/sprint-v{N}.md`
- User-facing wants → `US-###` in `.stories`
- Decisions and their trade-offs (incl. technology choices) → `ADR-###` in `.adr`
- How the system is actually built → `.spec/spec.md`
- Exploration notes, per-feature detail — anything that changes per sprint

## Editing

Living but scarce. A **content** edit is a strategic shift — vision, users, value proposition, principles, north-star, quality goals, non-goals, DoD — and pairs with an `ADR-###` carrying the why. Everything else is a typo fix in place. Either way, bump `last_reviewed`.

## Example

```markdown
---
last_reviewed: 2026-06-13
---

# Relay

## Vision
Every webhook a small team sends arrives exactly once — or they can see exactly why it didn't.

## Problem
Teams that emit webhooks rebuild the same delivery machinery — retries, signing, dead-lettering, receiver debugging — badly, on every project, and failures stay silent and unexplainable.

## Target users
Backend teams of 1–10 shipping APIs that must notify third-party receivers.

## Value proposition
Drop-in delivery with full per-event forensics, self-hosted: managed competitors offer delivery or visibility, never both without taking custody of payload data.

## Principles
- **Self-hosted over convenient** — payloads never leave the operator's infrastructure, even when a managed feature would be easier to build; the No-leak guardrail below is the measurable edge of this — it covers payload bytes, not delivery metadata, which Relay may relay through its own update channel.
- **Explainable over silent** — every delivery outcome is reconstructable from stored evidence; no fire-and-forget path exists.
- **Correctness over latency** — a delayed delivery beats a duplicate one; idempotency and ordering win ties.

## North-star metric
**Explained-delivery rate** — the share of events that either arrive (acknowledged 2xx) or carry a complete failure trace. It is the one number because both halves of the value proposition — delivery and forensics — degrade it when they slip.

## Quality goals
- **Delivery latency** — p95 enqueue-to-first-attempt must not exceed 2s at 1k events/min.
- **No silent failure** — every terminal event state writes a trace record; zero terminal states without one.
- **No leak** — payload bytes never cross infrastructure the operator doesn't run; checked at every storage and egress path.
- **Crash durability** — a node crash loses no accepted event (at-least-once across restarts).
- **Time to first delivery** — a fresh operator reaches first delivery from a single binary in under 15 minutes.

## Non-goals
- We will never store or relay payloads through infrastructure we operate.
- No general-purpose message queue — webhooks out, nothing else.
- No billing or seat logic in the core; monetisation lives outside this codebase.

## Definition of Done
A change is done when:
- Every acceptance criterion of its tickets passes on the code as it lands.
- The build-order Verify command passes on the landing tip.
- New behaviour carries tests; fixed behaviour carries a regression test.
- No Quality goal above regresses — measured at its stated threshold where a check exists, argued where not.
- `.spec/spec.md` reflects the change as landed.
- Any new dependency or externally visible contract change has an `ADR-###` recording the choice.

Outcome criteria (judged across the sprint, not one PR):
- The sprint's landed change leaves Explained-delivery rate no worse than its pre-sprint reading.
```
