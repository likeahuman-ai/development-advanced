# Stories Format Reference

`.stories/STORIES.md` holds the CURRENT set of wants — everything the product is meant to do, built or not. One file, no frontmatter. It is a **living current-relevant** document: it always describes what the product should do *now*, not the chronological record of everything it was ever asked to do. Git keeps the history; the file keeps the truth.

## Rate of change

Each artifact in this plugin earns its shape from how fast it changes and whether its history is load-bearing:

- The **ADR** (`.adr/ADR.md`) is append-only — past decisions are evidence and must never be edited away.
- The **Sprint Plan** (`.sprint/sprint-vN.md`) is cycle-versioned — each cycle freezes an immutable snapshot.
- **Stories** are neither. They are **living and current-relevant**, not append-only. A want that no longer applies is removed, not struck through or archived in place. Because the file carries no version and no status frontmatter, there is no `version:` / `status:` / `date:` header — nothing about Stories is versioned or staged.

## Two jobs

Stories do exactly two things, and nothing else:

1. **Alignment** — they let anyone ask "does the product satisfy our current wants?" The file is the canonical list of what the product is *for*.
2. **Sprint fuel** — the wants that are not yet built are the source `/sprint-plan` draws from to decide what a cycle should cover. (This is the *story backlog* — distinct from the *findings backlog* at `.sprint/backlog.md`, which holds deferred review findings. Two backlogs; this one is the wants.)

That is the whole purpose. Stories are not a status board, not a coverage tracker, and not a place to argue about how something gets built.

## Entry Template

```markdown
## {Epic name}

### US-{NNN}: {short title}

As a {role}, I want {capability}, so that {benefit}.

**Epic / Parent:** {epic name or parent US-### this rolls up to}
```

Rules for an entry:

- The **`As a {X}, I want {Y}, so that {Z}.`** sentence is mandatory and complete. The `so that` clause is required — a want with no benefit is not a story.
- **`{X}` names a specific role, not a bare "user".** It is a *user* story, so "user" is implied — the value is *which* role (it carries the permission, behavior, and security scope a downstream agent must honor). A single-actor product may use one named role; never leave it generic when roles differ.
- The **Epic / Parent** line is mandatory. Every story rolls up to an epic (or a parent story).
- Entries are **grouped under their epic heading** (`## {Epic name}`), so the file reads as a small tree of epics and their stories.
- **No status field.** Built-vs-unbuilt is not recorded here.
- **No acceptance criteria.** AC live on the tickets, not the story.

### Worked example

```markdown
## Onboarding

### US-001: First-run project detection

As a new user, I want the plugin to detect whether my folder is a
greenfield or existing project, so that the first cycle scaffolds the right
starting documents without me choosing.

**Epic / Parent:** Onboarding

### US-002: Resume an interrupted cycle

As a returning user, I want to pick up a half-finished cycle where I
left off, so that I don't lose the discovery work already captured.

**Epic / Parent:** Onboarding
```

## ID rules

- IDs are **stable, monotonic, zero-padded** three digits: `US-001`, `US-002`, `US-003`, ...
- An ID is **never reused.** Once `US-007` has existed, no future story may be `US-007`.
- **Removing** a story leaves a **permanent gap.** If `US-005` is deleted, the file jumps `US-004 → US-006`. The gap is correct — it tells you a want died and git records what it was.
- The **next ID** is always `highest-ever + 1`, not "highest currently present + 1." Track the high-water mark, not the live list.

## Mutability and removal

- Edit an entry **freely** — sharpen the wording, fix the role, correct the benefit. There is no immutability here.
- **Remove** an entry **only when the want itself dies** — the product no longer intends to do this. Never remove a story because it shipped. A built want stays in the file; that is how alignment works.

## Why no status and no acceptance criteria

- **Built-vs-unbuilt is derived, not stored.** `/sprint-refine` produces a spec **coverage view** at cycle close-out that maps stories against what the spec now describes. The answer to "is US-012 built?" comes from that derived view, never from a checkbox in this file. Storing it here would create a second source of truth that drifts.
- **AC belong on tickets.** A story is a want; a ticket is a unit of work with Given/When/Then. Putting AC on the story duplicates the ticket and rots the moment the ticket is refined.

## Conflict rule

When a new want contradicts an existing one:

- **Edit or remove the loser** in place. The file always reflects the current intent, so the contradiction must not survive in the file.
- **The WHY goes to an ADR.** The reasoning behind the reversal — why the old want lost — is a decision with load-bearing history, so it is recorded as an ADR entry, never as a comment or a struck-through line in `STORIES.md`. `/sprint-plan` performs this story-conflict detection and routes the rationale to the ADR.

## Optional INVEST check

For **new** entries only, you may sanity-check the story against INVEST — Independent, Negotiable, Valuable, Estimable, Small, Testable. This is a quality nudge while authoring, not a gate and not a recorded field. Never re-audit existing entries against INVEST; they earned their place.

## File Conventions

- **Single file:** `.stories/STORIES.md`. No per-epic files, no split.
- **Mandated from cycle one.** Even the greenfield first cycle creates `STORIES.md` alongside the Brief and `sprint-v1`.
- **Referenced, not reproduced.** The Sprint Plan's user-stories slice points at `US-###` and adds cycle-specific detail; it never restates the `As a … I want … so that …` sentence. The sentence lives here, once.

## What NOT to Include

- **No status** — no "done", "in progress", "todo", checkboxes, or coverage marks.
- **No acceptance criteria** — those are on the tickets.
- **No coverage tracking** — the built/unbuilt picture is the derived `/sprint-refine` coverage view, not a column here.
- **No decision rationale** — why a want changed or died goes to an ADR, not into this file.
- **No reused IDs** — gaps are permanent; never backfill a removed number.
- **No restating the sentence in the Sprint Plan** — the Sprint Plan references `US-###`; it does not copy the wording.

---

**Vocabulary:** *want* (a story's intent) · *epic* (the grouping a story rolls up to) · *high-water mark* (highest ID ever issued, used to pick the next) · *gap* (a permanently skipped ID left by a removed story) · *coverage view* (the derived built-vs-unbuilt mapping produced by `/sprint-refine`) · *loser* (the contradicted want that gets edited or removed when a conflict resolves).
