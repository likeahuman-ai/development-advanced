# labels-format

Shape of the sprint's **GitHub label set**: the seed Tickets creates before any issue or PR exists. Two parts — a **standing set** (always identical, created once per repo at 2.4.2) and a **per-sprint set** (derived per run at 2.4.3). Every label is created idempotently with `--force` so a re-run never errors.

**Load-bearing.** A missing label silently breaks a later `--add-label` (2.4.2). 2.4.2 runs before any issue/PR is created; if a name listed here is absent, issue creation (2.4.4), PR labelling (Build 3.4.2), and the state-machine transitions (4.6.4 · 5.2.2 · 5.2.3) fail downstream. The standing block below is the complete enumeration — every `gh label create --force` 2.4.2 must run, verbatim.

## Reference, never reproduce

A label is a **pointer**, never a copy of what it points at:

- `v{N}` → the sprint identity lives in `.sprint/sprint-v{N}.md`; the label is derived from the filename, never restates the plan's Goal.
- `epic:<name>` / `feature:<name>` → the grouping name comes from the breakdown's structure (2.3.2); the label slugs the name, never restates the epic's tickets.
- `type/*` / `complexity/*` → a ticket's own body owns its classification (`## Complexity`, `ticket-format`); the label mirrors it for forge-side filtering, never adds new fact.
- The state-machine labels (`needs-review` · `needs-refine` · `conflict-parked`) carry **forge state**, not artifact content — they say *where a PR is in the flow*, nothing about what it contains.

No description text restates a `.sprint`/`.brief`/`.adr` fact. *Artifacts are for AI, not humans.*

## State machine — the three live labels

A PR's flow state is **three labels only**: `needs-review → needs-refine → (closed on land)`. The PR closes automatically when it lands, so there is **no terminal label** — completion is a forge fact (the closed PR), not a label. **`sprint-complete` does not exist** — deliberately omitted; sprint completion = all PRs landed.

The transition table is the **shape of the flow state** — the artifact's single home for *which label sits on a PR at each phase*. (The step-by-step apply/remove orchestration is the consuming skill's process, not this format's.)

| State | Applied | Removed | Set / cleared by |
|---|---|---|---|
| `needs-review` | each new PR | when swapped to `needs-refine` | applied Build 3.4.2 · swapped Review 4.6.4 |
| `needs-refine` | when swapped from `needs-review` | at cleanup after land | applied Review 4.6.4 · removed Refine 5.2.3 |
| `conflict-parked` | when a land records a rebase conflict | when a later land of that PR completes | applied Refine 5.2.2 · removed Refine 5.2.2 (land completion — dissolved re-attempt or heal-resume) |

Invariants the transitions guarantee:

- `needs-review` and `needs-refine` are **never both present** — 4.6.4 swaps one for the other (no PR carries both at once).
- `conflict-parked` is **never removed at the park** — it comes off at 5.2.2 when a later land of that PR completes, whether the re-attempt's fresh rebase dissolved the conflict on its own (5.0.4's re-attempt route) or a paired session healed it at the halt. It is additive to the live label, not a replacement.
- Cleanup (5.2.3) removes the **flow-state label** `needs-refine`; `conflict-parked` comes off separately at the 5.2.2 land completion, not here — and cleanup **never** touches `v{N}`, `epic:*`, or `feature:*` (those are sprint identity, not flow state).
- `needs-review` is the discovery key for Review (4.1.1 — `gh pr list --label needs-review`); `needs-refine` is the discovery key for Refine (5.0.1).

## The emitted artifact — one block, standing then per-sprint

2.4.2 and 2.4.3 run in one Tickets pass, so the format is **one fenced `bash` block** with two comment-delimited regions in this fixed order: the **standing region** first (`# === standing (2.4.2) ===`), then the **per-sprint region** (`# === per-sprint (2.4.3) ===`). The standing region is verbatim from the seed below; the per-sprint region is the derived `gh label create` lines this run generates (rules below). One block, one run — no separate files, no second fence.

**Per-sprint region ordering — fixed, parse-stable.** Within the per-sprint region the lines emit in this exact order so a positional parse is stable: (1) the single `v{N}` line first; (2) then the grouping labels **grouped by epic** — each `epic:*` line immediately followed by **its own** `feature:*` lines (depth-first), never all epics then all features. The example below (one epic, one feature) is the degenerate single-epic case of this rule.

**Per-sprint labels accumulate — never cleaned up.** This block creates labels; it never deletes them. A sprint's `v{N}`/`epic:*`/`feature:*` labels **stay on their issues and landed PRs permanently** for traceability — cleanup never removes them (State-machine invariants). Each later sprint runs this same block: it **re-seeds the standing set** idempotently (`--force` — a no-op when present) and **adds its own** `v{N+1}`/grouping labels alongside the prior sprints' — the label namespace grows monotonically across sprints; nothing here regenerates or replaces an earlier sprint's set.

```bash
# === standing (2.4.2) — verbatim seed, identical every sprint ===
# --- state machine (3 live labels — no sprint-complete, by design) ---
gh label create needs-review     --force --color 0E8A16 --description "PR built, awaiting Review phase"
gh label create needs-refine     --force --color FBCA04 --description "PR reviewed, awaiting Refine phase"
gh label create conflict-parked  --force --color B60205 --description "Land rebase conflict — parked for paired human resolution"

# --- ticket classification: type/* ---
gh label create type/feature     --force --color 1D76DB --description "New user-facing behaviour"
gh label create type/fix         --force --color D93F0B --description "Corrects broken behaviour"
gh label create type/refactor    --force --color 5319E7 --description "Structural change, behaviour preserved"
gh label create type/chore       --force --color BFD4F2 --description "Tooling, deps, config — no product behaviour"
gh label create type/docs        --force --color 0075CA --description "Documentation / artifact-only change"
gh label create type/test        --force --color C2E0C6 --description "Test-only addition or change"

# --- ticket resource-cost: complexity/* (S/M/L, not time) ---
gh label create complexity/S     --force --color C5DEF5 --description "Small — narrow scope, low blast radius"
gh label create complexity/M     --force --color 7FB3E0 --description "Medium — moderate scope or blast radius"
gh label create complexity/L     --force --color 1B6CA8 --description "Large — broad scope, high blast radius"

# --- the build-order issue (single per sprint) ---
gh label create build-order      --force --color 000000 --description "The pinned build-order issue /sprint-build reads"

# === per-sprint (2.4.3) — derived this run (rules below); example shown ===
gh label create v7 --force --color D4A64A --description "Sprint 7"
gh label create "epic:onboarding" --force --color 5319E7 --description "Onboarding"
gh label create "feature:signup-form" --force --color BFD4F2 --description "Onboarding › Signup Form"
```

`type/*` and `complexity/*` are families — every member in the standing region above ships, no subset. The S/M/L cost mirrors a ticket's `## Complexity` (`ticket-format`); `type/*` mirrors the conventional-commit type the ticket's atomic commit will carry.

## Per-sprint region — generation rules

The per-sprint lines are **derived**, never fixed — the rules below define how each name, color, and description is produced from this run's inputs. Color and description are templated; only the name varies. The lines above (`v7`, the `epic:`/`feature:` pair) are an *instance* of these rules, not a fixed block.

### `v{N}` — sprint version

- **Derivation rule.** Take the `.sprint/sprint-v{N}.md` filename; `N` is the integer matched by `sprint-v(\d+)\.md`. The label name is the literal `v` + that integer. (`sprint-v7.md` → `v7`; `sprint-v12.md` → `v12`.) One `v{N}` per sprint.
- **Color** `D4A64A` (gold). **Description** the literal `Sprint ` + `N` (e.g. `Sprint 7`) — a pointer to the sprint, not a copy (reference rule above).

### `epic:<name>` / `feature:<name>` — grouping

- **Which groups produce a label.** Only the levels 2.3.2 kept: a flat breakdown creates **no** grouping labels; 2-level creates one `epic:*` per epic; 3-level creates one `epic:*` per epic **plus one `feature:*` per feature**. A collapsed degenerate level produces no label.
- **Completeness contract — one label per kept group, no exceptions.** Enumerate the kept structure **exhaustively**: every epic in the breakdown yields exactly one `epic:*`, and every feature under every epic yields exactly one `feature:*`. A count is never supplied separately — the breakdown's shape *is* the count. (A 3-level breakdown with 2 epics and 5 features across them yields exactly 7 grouping labels: 2 + 5. Omitting any feature contradicts the breakdown and leaves a queried epic with fewer features than it owns.)
- **Plan-owned epic tokens (the normal case).** When the plan carries an `## Epics` section, its `epic:<name>` token **is** the label name — used **verbatim, no transform** (sprint-format binds the token to this format's slug charset, so it is already label-safe). The slug rule below applies to `feature:*` display names and to legacy plans with no `## Epics` section.
- **Slug rule** — a pure character transform on the group's display name **verbatim**, forming the part after the `epic:` / `feature:` prefix. The display name is the source of truth: the slug **never expands, spells out, or rewrites a token** — an abbreviation in the name stays abbreviated (`Consent Mgmt` → `consent-mgmt`, not `consent-management`); the transform is reversible-by-character, not semantic. Steps, in order: lowercase · spaces → hyphens · strip punctuation (keep `a–z 0–9 -`) · collapse repeated hyphens · trim leading/trailing hyphens.
  - `Payments & Billing` → `epic:payments-billing`
  - `User Auth (v2)` → `feature:user-auth-v2`
  - `Consent Mgmt` → `epic:consent-mgmt` (no expansion — the display name owns spelling)
- **Parent recoverable from the label alone.** A flat label string loses the hierarchy, so the **parent epic lives in the `feature:*` description**: `<Epic display name> › <Feature display name>` (e.g. `feature:signup-form` under epic `Onboarding` → description `Onboarding › Signup Form`). An `epic:*` description is just its own display name. This makes feature→epic ownership recoverable from `gh label list` without reading any issue — the issues carry the same pair (2.4.4), but the linkage no longer lives only there.
- **Color** `epic:*` → `5319E7` (purple) · `feature:*` → `BFD4F2` (light blue).

```bash
# breakdown: epic "Payments & Billing" with feature "User Auth (v2)"
gh label create "epic:payments-billing" --force --color 5319E7 --description "Payments & Billing"
gh label create "feature:user-auth-v2"  --force --color BFD4F2 --description "Payments & Billing › User Auth (v2)"
```

## What each object carries

The grouping/identity labels the producer reads off the breakdown to know **what to generate** (the *applying* of labels to issues/PRs is the consuming skill's process, 2.4.4 · Build 3.4.2 — not this format):

- **Each ticket issue** → `v{N}` + its `epic:*` / `feature:*` grouping label(s) + (optionally) `type/*` + `complexity/*`.
- **The build-order issue** → `build-order` + `v{N}`.
- **Each PR** → `needs-review`, then the state machine above (the PR inherits no grouping labels; its tickets carry those).
