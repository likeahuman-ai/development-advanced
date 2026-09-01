---
name: sprint-tickets
description: "Turns an approved sprint plan into AI-ready GitHub Issues plus a pinned build-order issue, written to GitHub via gh — sized for the team's shared development trunk, team-scale parallel building, and the spec/ADR knowledge layer the tickets cite. Use when working on that trunk/team repo: a sprint plan is approved and needs implementation planning, or the user says 'break this down', 'create issues', 'make tickets', or 'turn this into tasks'. A solo, sequential, single-PR project belongs to the sibling development plugin instead."
argument-hint: "Path to the .sprint plan (optional — auto-detects the latest draft)"
---

# /sprint-tickets — Sprint Plan to Issues + Build Order

Phase 2 of the flow: turn the approved `.sprint` plan into AI-ready issues and a pinned build-order. Drive the sub-phases in order — 2.0 Phase setup → 2.1 Context → 2.2 Architecture → 2.3 Decomposition → 2.4 Issues → 2.5 Build order → 2.6 Phase close. Mostly autonomous: exactly one human gate (2.3.4), before any forge write.

Boundaries that hold for the whole phase:

- **Zero chain ops.** No commits, no pushes, no branch moves, no git write of any kind. The session's required vcs commands are **reads only** — `git remote -v` (2.4.1), plus, in with-`.spec` runs, one history read (`git log` against the spec's `last_updated`) computing the stale paths that fill the explorer prompt's stale-paths slot (2.1.1 — the explorers are Read/Glob/Grep-only and cannot query history themselves). vcs reads are inspections, never writes. The plan stays `status: draft`; the `built` flip rides the Refine spec-sweep commit (Refine 5.3.1, the run that completes the sprint), never this phase.
- **Forge through `gh` via Bash** — no MCP servers (an MCP catalogue front-loads and rots context; prefer on-demand CLIs). Every dispatch runs on a capable tier — Sonnet or Opus, never Haiku.
- **Dispatch, don't do.** Exploration belongs to `codebase-explorer`, design to `code-architect`. Consolidate their reports; never do their specialist work inline.

## Trust the artifact

The `.sprint` plan and `.adr` are approved decisions — gated at Plan 1.3.1, committed at Plan 1.3.2. Execute them as written: never re-validate, re-confirm, or re-present them for approval — a check whose normal outcome is "confirmed, proceed" is noise. The one sanctioned deviation path is 2.1.2's reconcile: the code reveals what the plan didn't account for → flag → `/sprint-plan`. Never patch the plan here, never re-commit it; if the user wants a plan change → `/sprint-plan`.

The `.spec` is different in kind — a claim about code, and code is ground truth. Verify it on contact (2.1.1): drift detection, not distrust.

**Initial request:** $ARGUMENTS

---

## 2.0 Phase setup

**Goal:** gather intent context — what to build (the `.sprint` plan + the upstream want/why). No code, no `.spec` — that's 2.1.

1. **Find the `.sprint` plan (2.0.1).** $ARGUMENTS carries a path → use it; else take the latest `.sprint/sprint-v{N}.md` with `status: draft`; if ambiguous → confirm with the user. **No draft exists → stop** — nothing to ticket; run `/sprint-plan`. The plan is already committed (Plan 1.3.2): trust it, never re-commit; if the user wants a change → `/sprint-plan`.

2. **Read the plan + upstream (2.0.2).** Read the `.sprint` plan **in full** — it is the contract. Pull forward the working inputs: the epic partition (the plan's `## Epics` section — the plan owns it; reuse it **verbatim** for the 2.2 architect fan-out, the 2.4.3 `epic:*` labels, and ticket Traceability), the `US-###` slice, scope, success metric, DoD-ref. An older plan with no `## Epics` section → derive the partition **once, here** (one epic per coherent delivery grouping of the `US-###` slice + Scope rows), record it in-session, and reuse that record verbatim everywhere the partition is consumed. Then read upstream, each skip-if-absent:
   - `.stories` — only the referenced `US-###` (selective — it can be large);
   - `.brief` — the DoD + quality goals;
   - `.adr` — **in full** (standing law; scarce + terse by the ADR bar — which decisions govern emerges during decomposition, not at planning).

---

## 2.1 Context

**Goal:** gather code context for writing tickets.

1. **Explore for context (2.1.1).** Check whether a `.spec` exists (skip-if-absent) — its presence selects the mode. Dispatch `codebase-explorer` agents (parallel), each briefed per `codebase-explorer-prompt` (`${CLAUDE_PLUGIN_ROOT}/skills/sprint-tickets/prompts/codebase-explorer-prompt.md`) — thorough, **one fresh exploration every run**: never reuse another session's map. Two modes:
   - **With `.spec`** — the spec scopes the start, **never the limit** (*direction from the session, discretion to the agent*): don't re-derive what it documents, but **verify on contact** — check any spec claim this sprint rests on against the code; mismatch → report (feeds 2.1.2). Paths with commits newer than the spec's last patch → re-explore. Always read the code on correctness-critical paths.
   - **Without `.spec`** — the full sweep: all three explorer modes (Architecture Mapping · Pattern Matching · Integration Analysis).

   Either mode: **surface the shared seams** — modules 2+ epics touch — they feed seam-ownership in 2.2.1.

2. **Reconcile (2.1.2).** Reconcile the gathered context against the `.sprint` plan + `.adr` (the approved intent):
   - the code reveals what they didn't account for (anti-pattern, blocker, false assumption) → flag → `/sprint-plan`;
   - else **`.sprint`/`.adr` win** — execute as written, proceed silently.

   A **spec mismatch** from 2.1.1 does **not** bounce to `/sprint-plan` — carry it as a note into the tickets' context; the map is corrected at the spec sweep (Refine 5.3.1).

---

## 2.2 Architecture

**Goal:** decide how to build it — *design before decomposition*: the approach is settled here, as its own act, before anything is sliced.

1. **Prepare architect prompts (2.2.1).** Per epic, distil 2.0 + 2.1 into one succinct dispatch prompt per `code-architect-prompt` (`${CLAUDE_PLUGIN_ROOT}/skills/sprint-tickets/prompts/code-architect-prompt.md`): scope (`.sprint`) · the governing `.adr` records · the module map · **the shared seams, each with a single-owner assignment** · the `.spec` slice (if present). Pointers + boundaries, not a code dump.

2. **Design per epic/feature (2.2.2).** Dispatch `code-architect` agents (parallel), one per epic/feature; each deep-reads its own slice. The design lives in the agent's **report** — no design-doc file is produced or persisted. Seam discipline rides in each prompt: consume owned-elsewhere interfaces as given; a needed change → flag, don't fork the seam; a *new* seam's consumer designs against the assigned owner's expected shape (parallel dispatch — the owner's design isn't back yet). Mismatches surface at 2.3.1's coherence check.

   **Parallelisation lens.** When two units would share code, the design's default is to place that code in its **lowest common owner** (core / runner) so both consume it as peers — never to have one unit *reuse* another's, which manufactures a hard dep and serialises work that could fan out in one wave. A reuse-coupling is a real dependency **only** when the shared code genuinely can't be factored down; otherwise factor it down. Genuine *layering* (a surface that imports a core) stays a real dep — that is the dependency that legitimately divides into a stack at 2.5.3; manufactured serialisation is the one to design away.

---

## 2.3 Decomposition

**Goal:** consolidate the architects' tickets into one coherent, right-sized, approved backlog.

1. **Assemble the draft ticket list (2.3.1).** Collect the architects' reports — each carries one epic's design as ticket-sized units: objective, hard deps, AC, S/M/L complexity, `US-###` refs. Combine into **one cross-epic draft ticket list**, in-session (it becomes Issues at 2.4.4). Check interface coherence: no seam defined two ways, no seam-ownership flag left unresolved, and **no false-serial reuse-coupling** — a hard dep that exists only because one ticket reuses code another *introduces* (not genuine layering) is a parallelisation miss: push the shared code down to a common owner (core / runner) so both tickets fan out in one wave (the 2.2.2 lens), and keep the dep only when it truly can't be factored down. The hard-dependency + complexity notes pass forward to 2.5. No per-file attribution is collected anywhere — file overlap is no input under worktrees. Then the **per-ticket clarity audit** — the executor won't notice what's missing, so implementability is asserted at write time, per draft ticket: every reference claiming an **existing** seam (a `file:line` anchor, a named existing symbol) exists in the tree (grep it — a named existing seam that doesn't is a BLOCKED implementer later); paths and symbols this ticket creates (`new:` anchors) or consumes from a `Blocked by:` ticket are exempt — for those, check the creating ticket is named · every **behaviour criterion** (its Then/`shall` names runtime behaviour — a state change, a response, an emitted effect — not a static property the gate script checks) is **post-land observable**: the session names, in-session, the command or corridor suite that will observe it after land — a viability judgement, not ticket content; the AC text never carries a command and nothing is recorded (observation is corridor territory, outside the loop: the loop's gates check form only, so no in-loop run observes an AC). A criterion the session can name no observer for — today's repo or the corridor's next suite — is a wish, not a check: tighten its observable in the draft list, or route it back to the architect's report · **no unresolved choice hides in prose** ("or", "either", "TBD", "depending on" in a requirement is a decision the implementer will be forced to make — resolve it here or route it back to the architect's report). A ticket that fails the audit is fixed in the draft list before 2.3.3 presents it.

2. **Decide structure (2.3.2).** Mirror the breakdown's natural shape; collapse degenerate levels: 1 grouping → flat (labels only) · N groupings → 2-level (epic → tasks) · epics with sub-features → 3-level (epic → feature → tasks). Collapse any single-child level. No count threshold. Structure serves **traceability** (`US-### → issue → PR`, Refine 5.3.2) **+ human navigation** — no agent reads a parent issue → when in doubt, flatter.

3. **Present breakdown (2.3.3).** Present the grouped breakdown to the user — groups at the depth 2.3.2 chose, each ticket with title, complexity, and hard deps, plus anything the user should weigh.

4. **Gate — approve the breakdown (2.3.4).** The user must approve the breakdown **before any forge write**. Changes → amend the draft list, re-present. No `gh` write runs until this gate passes. The phase's only gate — it sits on acceptance of the breakdown's *content*, never on a vcs/forge op: the human approves the artifact, and the forge writes that publish it follow autonomously.

---

## 2.4 Issues

**Goal:** turn the approved breakdown into forge objects.

1. **Find the target repo (2.4.1).** `git remote -v` → pin the repo every `gh` write in this phase targets (guards the multi-remote case); if no remote → stop — the Plan 1.0.2 bootstrap should have created it; route back to `/sprint-plan`. One of the phase's two sanctioned vcs reads — the boundary block at the top owns the list.

2. **Seed the standing labels (2.4.2).** Seed the full standing set `labels-format` (`${CLAUDE_PLUGIN_ROOT}/skills/sprint-tickets/formats/labels-format.md`) enumerates — the sprint state machine (`needs-review` · `needs-refine` · `conflict-parked`) + the `type/*` and `complexity/*` taxonomies + `build-order`. Every create is `gh label create … --force` (create-or-update, idempotent: a no-op on an established repo, the full seed on a fresh one). Load-bearing — a missing label silently breaks a later `--add-label`.

3. **Seed the per-sprint labels (2.4.3).** Create `epic:*` / `feature:*` from the breakdown's groups — only the levels 2.3.2 kept — plus `v{N}` from the plan filename (`.sprint/sprint-v{N}.md` → `v{N}`); `--force`.

4. **Create the issues (2.4.4).** One `gh issue create` per ticket — body per `ticket-format` (`${CLAUDE_PLUGIN_ROOT}/skills/sprint-tickets/formats/ticket-format.md`), passed with `--body-file`, **and labelled at creation**: `--label` carries `v{N}` plus the ticket's kept grouping label(s) (`epic:*`/`feature:*`, per labels-format's "What each object carries"; `type/*`/`complexity/*` attach only on explicit user request — the ticket body owns its classification, so by default they stay seeded-but-unattached). Parents before children at the depth 2.3.2 chose, in dependency order, threading the `#refs` (`Part of #N` · `Blocked by #N`). **Parent issues have their own shape** (ticket-format fits leaf tickets only): title `[Epic] <group name>` — the literal marker Build 3.4.3 and Refine 5.3.4 close by — with a minimal body (one line naming the epic's scope; **the child record is the children's own `Part of #<epic>` threading** — GitHub renders those cross-references on the parent automatically, so the parent body is never edited after creation) and labels `v{N}` + its own `epic:<name>`. No milestone — `v{N}` is the sprint tie. Write discipline:
   - **One create per Bash call — never a generated batch script.** Sequential writes dodge GitHub's secondary rate limit and keep each failure legible: if #9 fails, #1–8 exist and the stop point is visible.
   - Write each body to a temp file first (bodies are markdown full of backticks and `$` — a heredoc mangles them), then point `--body-file` at it.
   - Read the returned `#N` from each create's output and write it **literally** into the next dependent body — shell variables don't survive between calls; if a create returns no number → stop and report — never create a child referencing a parent that doesn't exist.

5. **Present summary (2.4.5).** Present the created issues — grouped, with `#`numbers + their GitHub URLs.

---

## 2.5 Build order

**Goal:** write the self-contained build-order issue `/sprint-build` executes — Build trusts it as authoritative and never recomputes (Build 3.1.3).

1. **Collect per-ticket data (2.5.1).** For each ticket, pull exactly two things from its architect's report: **hard dependencies** — the tickets whose *created* artefacts this one consumes (it won't build or run without them; value must flow, nothing else counts) — and its **S/M/L complexity**. Rewrite the dep references as the created `#N`s → the input table for 2.5.2. Write-sets and soft deps are dropped — worktrees removed the file-clobber and implementers work isolated worktrees the session collects, so nothing needs per-file attribution.

2. **Compute hard-dep waves (2.5.2).** Sort the tickets by dependency — every ticket after the tickets it depends on (a topological sort), layered: a **wave** = every ticket whose deps all sit in earlier waves. **Hard deps only — no file-disjointness checks of any kind** (worktree isolation absorbs file overlap; a rare same-line overlap surfaces at Build's collection as a controlled halt, healed there — a coupling signal, not something to pre-empt); if the sort stalls (a dependency cycle) → reclassify a mis-tagged dep, or combine truly co-dependent tickets into one. Dep-changes need no special handling — the lockfile regenerates at Build authoring (Build 3.2.4). → `## Parallel Waves`

3. **Group PRs (2.5.3).** Group the tickets into PRs by **coupling** — a shared runtime boundary: they ship one behaviour, and split, neither PR reviews alone — never by line count. And **dependency-closed**: a group includes everything it hard-depends on, or depends only on what's already on `development`. Mark each group **peer** or **stacked**: a **peer** depends only on `development` — independently reviewable *and* landable in any order; a **stacked** group hard-depends on another group *in this sprint* — record `stacked on: <slug>` so Build partitions it `--base` that group (3.3.1) and Refine lands base-first (5.x). **Prefer peers** — the 2.2.2 parallelisation lens removes *false* couplings, so only genuine layering (core → surfaces) stacks; collapse a group into what it depends on only when splitting it out buys no review value. → `## PR Grouping`

4. **Decide scope (2.5.4).** State the build scope **as a decision, not a menu** — e.g. "build all N; #X is stretch (build only if the wave has room) unless told otherwise." Leaves Build no question to ask. → `## Scope`

5. **Write verify (2.5.5).** Write the `## Verify` entries — **written once here, never edited after**:
   - **provision** — one real, runnable, repo-matched command (detect the repo's own tooling — e.g. `pnpm install --frozen-lockfile`), install/sync project deps; what a fresh worktree runs first. Never a placeholder.
   - **verify** — the literal promise `./scripts/t0.sh` — **never a checker command string**. The script does not exist yet: Build makes the promise true as its first act — it generates the sprint's gate script from the repo's tooling, proves it, and commits it before any ticket is built. Writing checker invocations here (typecheck, compile, test commands) is the drift this promise exists to prevent: the gate's content has one owner, the proven script, and this section only names it.

   Checks the loop's gates deliberately do not run are recorded as **corridor-territory lines** — `deploy-deferred:` (suites that only run post-deploy) and `gate-excluded:` (an in-scope check deliberately left out) — behaviour testing that belongs outside the loop, kept as explicit records so an omission is never ambiguous silence. Consumers read the entries verbatim — the complete field → consumer index is single-sourced in `build-order-format`'s Field reference. → `## Verify`

6. **Create the build-order issue (2.5.6).** `gh issue create` per `build-order-format` (`${CLAUDE_PLUGIN_ROOT}/skills/sprint-tickets/formats/build-order-format.md`) — the body carries exactly the four sections written above: `## Parallel Waves` · `## PR Grouping` · `## Scope` · `## Verify`. Labels: `build-order` + `v{N}`. **No gate** — the build-order is deterministic from the approved breakdown; 2.3.4's acceptance covers it, and a forge write is never gated (gates sit on content acceptance, never on a vcs/forge op).

7. **Pin the build-order issue (2.5.7).** `gh issue pin` — the pin is how a fresh `/sprint-build` session finds its work. Build 3.4.3 unpins on retire — GitHub caps pins at 3, so the slot must come back; if the pin fails on the cap → list the pinned issues, unpin a stale `build-order` if one exists, else surface the three pins to the user.

---

## 2.6 Phase close

**Goal:** hand off to build.

1. **Handoff (2.6.1).** Summarise the created issues (grouped, `#N`s) + the pinned build-order. Recommend running `/sprint-build` in a **fresh session** — the pinned build-order carries the handoff state; nothing from this conversation is needed (*phases don't share a session — artifacts are the bridge*).
