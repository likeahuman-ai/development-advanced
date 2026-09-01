# stories-format

Shape of `.stories/STORIES.md` — the single living file of user-facing wants, one entry per want, grouped by epic, fronted by an ID ledger. The file always reads as what the product should do *now*: edits sharpen a want, removals kill a dead one, git keeps the history. Unbuilt wants are the pool `/sprint-plan` draws from to decide what a sprint covers; built wants stay as the alignment record. No frontmatter, no versions, no status field — current-relevant, not append-only.

## File

`.stories/STORIES.md` — one file from the first sprint on. Wants live here once; everything downstream cites a story by `US-###` and never copies the narrative out.

## Template

```markdown
# User Stories

## ID ledger

- Next: US-{NNN}
- Removed: {US-### list, or "none"}
- Gaps: {US-### list, or "none"}

## {Epic name}

### US-{NNN}: {short title}
As a {role}, I want {capability}, so that {benefit}.
**Epic / Parent:** {epic name | US-### this rolls up to}
```

A narrative whose `{role}`/`{benefit}` was not stated at issue carries a leading `~` (`~ As a {role}…`): the want is real, those clauses are a placeholder, not a recovered fact (`## Field rules`).

## Example

```markdown
# User Stories

## ID ledger

- Next: US-007
- Removed: US-003
- Gaps: US-004

## Onboarding

### US-001: Magic-link sign-in
As a new member, I want to sign in with an emailed magic link, so that I never manage a password.
**Epic / Parent:** Onboarding

### US-002: Resume where I left off
As a returning member, I want the app to open on my last unfinished lesson, so that I don't re-navigate the course tree every session.
**Epic / Parent:** Onboarding

### US-005: Skip the welcome tour
~ As a member, I want to dismiss the welcome tour permanently, so that repeat visits start at the content.
**Epic / Parent:** US-002

## Billing

### US-006: Download invoices
As an account admin, I want to download past invoices as PDF, so that I can file expenses without contacting support.
**Epic / Parent:** Billing
```

Reading the ledger against the live entries recovers every below-`Next` ID's state with no entry scan: US-001/002/005/006 live; US-003 `Removed` (want died, reasoning in `.adr`); US-004 a `Gaps` skip (issued-or-reserved, then vacated for reasons not a want-death — never to be backfilled); nothing else exists below `Next: US-007`. `US-005`'s leading `~` marks its role/benefit a placeholder — the want is live, but a consumer must not recover "member" as an authored permission scope.

## Field rules

- `## ID ledger` — the first section, exactly three lines, the recoverable source for every ID-provenance fact (derived state pinned, never inferred from the live max). Each `US-###` below `Next:` is in exactly one of three states; the ledger plus the live entries decide which, no entry scan:
  - **`Next:`** — the next ID a new story takes: the high-water mark + 1, where the high-water mark is the highest ID *ever issued or reserved*, not the highest currently present. Issuing the next story bumps `Next:`. IDs are assigned in order, so a new story may also take a `Gaps:` skip only by an explicit decision to backfill — by default the gap stays permanent (see `Gaps:`).
  - **`Removed:`** — every `US-###` retired because **the want died** (the product no longer intends it), ascending, or `none`. A `Removed` ID asserts a known cause; the reasoning lives in `.adr`, never here.
  - **`Gaps:`** — every `US-###` below `Next:` that is **neither live nor `Removed`**: an ID that was issued-or-reserved and then vacated for a reason that is *not* a want-death (a misnumber, a reservation never filled, a provenance no longer on record), ascending, or `none`. A gap is a permanent skip — a later want never backfills it absent an explicit decision. This is the distinct third bucket: it carries "skip of non-want-death (or unknown) provenance," which `Removed:` must not be made to assert.
- `## {Epic name}` — groups stories under one heading, by a deterministic rule: a story belongs to the epic naming **the user-facing surface its `{capability}` clause acts on** — and when that clause names more than one, the smallest. The `{benefit}` clause is never an input to epic placement (it can name an outward surface the capability never touches). So a story whose capability acts on stories but whose benefit is sharing/reporting belongs to the capability's surface (Story Management), not the benefit's. An epic is a grouping handle here only; ticket-side epic structure and labels are owned at Tickets.
- `### US-{NNN}: {short title}` — one heading per story, ID first, so a consumer holding a referenced `US-###` greps the heading and reads selectively (the file can be large). The title is a handle, not the want.
- Narrative line — mandatory, complete, one sentence: `As a {role}, I want {capability}, so that {benefit}.` The `{capability}` clause is always an authored, recoverable fact (what the want is). `{role}` (permission scope) and `{benefit}` are recoverable **only when authored**: if either was not stated at issue, the line leads with `~` and carries a best-guess placeholder for the unstated clause(s) — keeping the sentence complete without asserting an invented permission scope as fact. A `~` line means: trust `{capability}`; treat `{role}`/`{benefit}` as placeholder until authored. Never abbreviate to a fragment; never drop the `~` to make a placeholder read as authoritative.
- `**Epic / Parent:**` — mandatory. The epic name, or the `US-###` this story rolls up to (a sub-want still sits under its epic heading; the rollup is this line).
- Nothing else per story — no status, no acceptance criteria, no estimates, no coverage marks (each owned elsewhere — see the table below).

## Order

Epics in introduction order (first issued first); stories **ascending by `US-###` within each epic**. Document order is therefore grouped, not globally monotonic — the `## ID ledger` field rule owns the next-ID and gap facts, so no consumer scans entries to recover them.

## ID rules

The `## ID ledger` field rule owns the ledger's semantics (the three states, `Next:` as high-water mark + 1, gaps permanent). These are the per-ID handling rules:

- `US-` + zero-padded three digits: `US-001`, `US-012`, `US-104`.
- Stable, never reused: issuing takes the ledger's `Next:` and bumps it.
- Removing a want-died story: delete the entry, add its ID to `Removed:`, `Next:` unchanged.
- Vacating an ID for any non-want-death reason: delete the entry (if present), add its ID to `Gaps:`, `Next:` unchanged.

## Mutability

Living and current-relevant — not append-only, not versioned. The file always describes the product's intent now; git keeps what it used to say.

- Edit freely — sharpen wording, correct the role, fix the benefit. Authoring a placeholder clause for real drops its leading `~`. The file keeps the truth.
- Remove a story only when the want dies — the product no longer intends it. Delete the entry whole; no strikethrough, no archive section.
- Built wants stay. Shipping a story never removes it — the file states what the product should do, built or not. Built-vs-unbuilt is *derived* at the coverage view, never marked here.
- When a new want contradicts an existing story, the loser is edited or removed in place. The reversal's reasoning never lives in this file — it lives in `.adr` as an `ADR-###`. Stories is the *current intent* layer; `.adr` is the *reasoning* layer.

## Owned elsewhere — never in this file

| Fact | Lives in | Referenced as |
|---|---|---|
| Built / shipped status | derived at the coverage view: story → issue → PR → spec coverage | `US-### → #N → PR → .spec/spec.md#anchor` |
| Acceptance criteria | tickets | `#N` |
| Sprint scope — which stories a sprint covers | the sprint plan's `US-###` slice | `.sprint/sprint-v{N}.md` |
| Reversal / trade-off reasoning | decision records | `ADR-###` |
| Definition of Done, quality goals | the brief | `.brief/brief.md#definition-of-done` |

Downstream cites a story by `US-###` only — the sprint plan's slice, ticket metadata, architects' reports, commit trailers (`Story: US-###`).
