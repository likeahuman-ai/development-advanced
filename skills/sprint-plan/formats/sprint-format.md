# Sprint Plan format

Shape of `.sprint/sprint-v{N}.md` — the sprint's versioned objective, one file per sprint. Written at Plan 1.2.6. Must parse as a YAML frontmatter block + Markdown body. The body is **frozen once the plan is accepted** (Plan 1.3.1); `status:` is the only field that ever mutates afterwards.

A Sprint Plan states *what this sprint delivers and why* — never how at code level (`.spec/spec.md#anchor` and `ADR-###` carry the technical detail), never in-sprint progress (Issues carry that). Every fact owned elsewhere appears as a reference, never a copy: `US-###`, `.brief/brief.md`, `.spec/spec.md#anchor`, `#N`.

## Frontmatter

```yaml
---
version: 3               # sequential integer, no gaps; derives the v{N} label and feat/sprint-v{N}(-<g>) chain names
status: draft            # the ONLY mutable field — see lifecycle
date: 2026-06-13         # creation date, ISO; never updated
author: tim              # the human who owns the sprint; a bare name. An agent-run session with no human owner writes Session Agent — never invent a human
previous: sprint-v2.md   # prior sprint's file; null for v1
---
```

### `status:` lifecycle

| Value | Means | Flipped by |
| --- | --- | --- |
| `draft` | active — covers planning and all building-against, until the sprint's final PR lands | set on creation, Plan 1.2.6 |
| `built` | implementation complete — every `feat/sprint-v{N}(-<g>)` PR landed on `development` (a forge fact; this field annotates it) | Refine 5.3.1, riding the spec-sweep commit (the run that completes the sprint); irregular close (all PRs landed yet still `draft`) → the Plan 1.0.3 backstop, riding the next sprint's `docs(plan)` bundle |
| `archived` | superseded — a newer draft exists | Plan 1.0.3 cascade, riding the `docs(plan)` bundle |
| `abandoned` | draft cancelled before build (user-driven, rare) | Plan 1.0.3 |

- At most **one `draft`** exists across `.sprint/` at any time (Plan 1.0.3 enforces).
- No `released` (that is plugin `.prd/` lifecycle, not Sprint Plans). No `planned` / `complete` — sprint completion is the forge fact of landed PRs, never duplicated here.
- A status edit is an **annotation, not a change** — it never earns its own commit; it rides an existing one (which, per the `Flipped by` column above).

## Body sections — in order, all required unless marked optional

**Goal** — the sprint's pass/fail criterion in full; what 5.3.2 checks the landed PRs against, so every condition that decides pass/fail lives here (none demoted to Scope/DoD/stories). Two forms:

- **Single-condition** (default): `This sprint succeeds iff <one objective>.` One sentence, binary pass/fail. An "and" buried in one objective signals scope overflow — split or cut.
- **Multi-condition** (only when the objective genuinely couples 2–3 conditions that pass or fail together): a lead sentence naming the headline condition — the one the Success metric measures — then a bulleted acceptance list, every condition binary:
  ```
  This sprint succeeds iff <headline condition>, with:
  - <condition 2>
  - <condition 3>
  ```
  The headline is the metric-bearing condition (ties Goal ↔ Success metric); the bullets are the rest of the pass/fail set. Past three bullets it is scope overflow — split the sprint.

No condition that decides pass/fail may live only in Scope or DoD — Scope bounds *what's touched*, the Goal states *what must be true to pass*.

**Non-goals** — short bullets; what the sprint deliberately omits.

**Solution** — one non-technical paragraph; the highest-level delivery statement. No architecture, no filenames, no jargon.

**User-stories slice** — reference-only bullets of `US-###` IDs from `.stories`; never restate the story sentence. Each carries sprint-specific detail: which part this sprint delivers.
Form: `` `US-012` — sprint detail: … ``

**Epics** — the sprint's delivery partition, **owned here** and consumed verbatim by Tickets (2.0.2, which owns the consumer list — the partition is decided once, at plan time, never re-derived per run). One bullet per epic: `` `epic:<name>` — the `US-###` / Scope rows it groups ``. The `<name>` is the literal forge-label token, written **already label-safe** in labels-format's slug charset (lowercase `a–z 0–9 -`); Tickets copies it byte-for-byte with no transform, so the plan's token and the forge label are always the identical string. A single-epic sprint states its one epic — the section is never omitted (whether the breakdown then flattens is Tickets' 2.3.2 structure call, not an absent partition).

**Scope** — exactly ONE in/out table, both columns populated. Never a separate out-of-scope section.

```
| In scope this sprint | Out of scope this sprint |
| --- | --- |
| ... | ... |
```

**Architecture** — shape of change only, 2–3 lines: how the sprint alters the system, pointing into the Spec by anchor (`.spec/spec.md#anchor`) and to governing decisions in `.adr`. Reference a decision by its real `ADR-###` when it carries a number; an unnumbered decision is referenced by title (`.adr` "<decision title>") — **never invent a number** to satisfy the form (a fabricated `ADR-###` sends a downstream consumer following the anchor to the wrong or nonexistent record). NO directory tree, component catalogue, data-flow, or integration-point list — those live in `.spec`.

**Success metric** — ties this sprint's measurable target to the `.brief` north-star. Two distinct things, so name both: the north-star (the `.brief`-owned goal, referenced not restated) and **this sprint's concrete metric** (the measurable proxy the Goal's headline condition moves). When the north-star *is itself* a single number, the two collapse — say so by naming one metric for both roles. Pick the form the metric's state demands:

- **Move an existing metric:** `North-star (<.brief goal, defined in .brief/brief.md>) — this sprint's metric: <metric name>, cut from X to Y.`
- **Establish a first-time baseline** (no prior X to move from): `North-star (<.brief goal, defined in .brief/brief.md>) — this sprint's metric: <metric name>, establish baseline at <target>.`
- **They collapse** (north-star is the number): `North-star (<metric name>, defined in .brief/brief.md): cut from X to Y this sprint.`

If the metric carries a stated tracking horizon beyond this sprint, append it — `; tracked for <window> post-ship.` — never drop the window (it is a stated fact with no other home in this format).

**Timebox** — one line, duration only.

**Definition of Done** — an **anchored** reference to the canonical DoD: `.brief/brief.md#<anchor>` (the heading anchor, never a bare file path — a consumer resolves to the exact DoD heading, not the whole file); never restate its text. Add only sprint-specific exit conditions beyond the canonical DoD, if any exist.

**Dependencies & Risks** *(optional — include only when real)* — three-column table: `Dependency / Risk` · `Impact` · `Tracking Issue` (`#N` when a tracked issue already exists; `—` otherwise — the row itself is the **permanent** tracking record; the plan file is never edited after 1.3.2, so no later phase backfills the cell — a `/sprint-plan` revision may link an issue when one exists; never fabricate a number). Never park unmanaged risk text outside this table.

## Never include

- Full architecture block: directory tree, component list, data-flow diagram, integration list (`.spec` owns these)
- File tables (Source / Detailed-changes / Migration / Verification / History)
- A standalone "Out-of-Scope" section (the scope table's right column is the only home)
- A multi-row success-metrics matrix (one north-star, one target)
- User/System-Flow, Testing Strategy, Privacy & Security, or Rollback Plan sections (tickets' AC, `.spec`, and `.adr` own these)
- In-sprint progress — done / in-progress / blocked (Issues own status)
- Restated story sentences, DoD text, or Spec content (reference only: `US-###` · `.brief/brief.md` · `.spec/spec.md#anchor`)
- Open `[NEEDS CLARIFICATION: …]` markers (all resolved before the draft is accepted)
- Implementation code, ever

## Example

```markdown
---
version: 3
status: draft
date: 2026-06-13
author: tim
previous: sprint-v2.md
---

# Sprint v3 — Webhook delivery retries

## Goal

This sprint succeeds iff a failed webhook dispatch is delivered within 5s, with:
- retries spanning a 2-hour window before dead-lettering
- dispatch signed with the rotated secret, verified end-to-end

## Non-goals

- No manual replay controls or retry UI
- No multi-region dispatch

## Solution

Failed deliveries stop disappearing silently: the platform now retries them on its own and sets aside the ones that keep failing, so integrators receive every event without anyone watching a queue.

## User-stories slice

- `US-007` — sprint detail: automatic retry with backoff; the dead-letter notification email ships next sprint
- `US-012` — sprint detail: dead-letter queue persisted and queryable, full slice

## Epics

- `epic:retry-pipeline` — `US-007`; retry/backoff + delivery-outcome counter scope rows
- `epic:dead-letter-store` — `US-012`; dead-letter persistence scope row

## Scope

| In scope this sprint | Out of scope this sprint |
| --- | --- |
| Retry with exponential backoff | Manual replay from dead-letter |
| Dead-letter persistence after max attempts | Dead-letter notification email (`US-007` remainder) |
| Delivery-outcome counter | Multi-region dispatch |

## Architecture

A `RetryQueue` slots between dispatcher and worker pool — see `.spec/spec.md#runtime-view`. Dead letters persist through the existing store (`.spec/spec.md#data-model`); no schema change. Decision context: `.adr` "async queue selection", `.adr` "caching strategy" (both unnumbered).

## Success metric

North-star (delivery latency, defined in `.brief/brief.md`): establish baseline at <2s local / <5s staging this sprint; tracked for 2 weeks post-ship.

## Timebox

1 week.

## Definition of Done

Canonical DoD per `.brief/brief.md#definition-of-done`. Sprint-specific addition: a forced-failure dispatch demonstrably lands in the dead-letter store.

## Dependencies & Risks

| Dependency / Risk | Impact | Tracking Issue |
| --- | --- | --- |
| Upstream signing-key rotation lands mid-sprint | Retried dispatches could fail signature checks | #58 |
```
