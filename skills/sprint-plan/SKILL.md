---
name: sprint-plan
description: "Turns intent into an approved, committed Sprint Plan — plus ADRs and the living Brief and Stories — on the team's shared development trunk, where sprints are built team-scale in parallel against the spec/ADR knowledge layer. Use when planning team work in that trunk/team environment: the user has an idea for what to build, says 'I want to build', 'let's plan', 'I have a project idea', wants to start a new sprint, or wants to resume an unfinished sprint plan. A solo, sequential, single-PR project belongs to the sibling development plugin instead."
argument-hint: "Brief description of what to build (optional)"
---

# /sprint-plan — Intent to an Approved Sprint Plan

Phase 1 of the flow: turn intent into an approved, committed `.sprint` plan (+ `.brief`/`.stories`/`.adr`). Drive the sub-phases in order — 1.0 Sprint setup → 1.1 Discovery → 1.2 Author → 1.3 Phase close. Three human gates, all on **content acceptance** (1.1.3 discovery summary · 1.2.5 ADR fidelity · 1.3.1 the plan); a gate approves an artifact's content, never a vcs op — the commit that publishes it follows autonomously. Never gate a commit or a push.

Four artifacts, separated by rate of change:

- `.brief` — the living product charter; slowest-changing, written once, edited scarcely (`brief-format`)
- `.stories` — the living set of wants, `US-###` IDs (`stories-format`)
- `.adr` — the append-only decision log, `ADR-###` (`adr-format`)
- `.sprint/sprint-v{N}.md` — this sprint's versioned objective, the primary write target (`sprint-format`)

`.spec` is read-only here — it is a claim about code, patched by Refine (5.3.1), never by this skill.

This skill's formats (`brief-format` · `stories-format` · `adr-format` · `sprint-format`) live at `${CLAUDE_PLUGIN_ROOT}/skills/sprint-plan/formats/<name>-format.md`; dispatch prompts at `${CLAUDE_PLUGIN_ROOT}/skills/sprint-plan/prompts/<name>-prompt.md` — the filename keeps the `-format`/`-prompt` suffix. `commit-format` is owned by Build: `${CLAUDE_PLUGIN_ROOT}/skills/sprint-build/formats/commit-format.md`.

Boundaries that hold for the whole phase:

- **All vcs through the session's own git.** The session runs every git command itself; `gh` drives the forge through Bash — on-demand CLIs, never MCP servers (an MCP server front-loads its whole tool catalogue and rots context).
- **No push anywhere in this phase** (1.3.3). Plan is local-only; publication is Build-close (3.4.1). Sole exemption: 1.0.2's bootstrap scaffolding (repo create · trunk seed · protection) — scaffolding, not flow history.
- **The branch stays local.** 1.0.5 cuts `feat/sprint-v{N}` off the fetched trunk — a local branch, deterministic name; no remote ref exists until publication (Build 3.4.1).
- **Every dispatch runs on a capable tier** — Sonnet or Opus, never Haiku (a misread at a gate or cull corrupts everything downstream).
- **Commits carry `Assisted-by: <agent> <model>`** — never `Co-Authored-By:` (a model can't hold copyright).
- **Dispatch, don't do.** The big exploration belongs to `codebase-explorer` agents; the session steers the conversation, gates the content, and authors the commits.

**Initial request:** $ARGUMENTS

## Preflight

Before anything else, verify the two tools the phase rests on:

- `git --version` — present; missing → **stop** with install guidance, re-run once installed
- `gh auth status` — authenticated; not → **stop**: `gh auth login`, then re-run

---

## 1.0 Sprint setup

**Goal:** ready the checkout, repo, and plan version.

### 1.0.1 List existing context

List what exists — **no reads**:

- file artifacts — which of `.brief` / `.stories` / `.spec` / `.adr` / `.sprint` exist
- repo — git (`.git` present)? remote?
- → greenfield or existing project

Existence only — content (versions, statuses, history) is read fresh by the step that needs it (`.sprint` statuses → 1.0.3).

### 1.0.2 Prepare the repo

Bootstrap on first contact — one homogeneous bundle (same kind of acts, each act visible), each a no-op when already done:

- if no repo → `git init`
- git identity if unset (one-time per machine): `git config user.name` / `git config user.email` empty → set them (ask the user; never invent)
- if no remote → `gh repo create <owner>/<repo> --private --source=. --remote=origin` (provisional bootstrap)
- if no `development` anywhere (greenfield) → **seed the trunk**:
  ```bash
  git checkout -b development
  git commit --allow-empty -m "chore: seed development"
  git push -u origin development
  ```
  Bootstrap scaffolding, exempt from the no-push doctrine (like `gh repo create` itself) — 1.0.4/1.0.5 need the trunk to exist.
- protect `development` — **require linear history + block force pushes**, never "require a pull request" (Refine 5.2.2 pushes the trunk directly); applies once the trunk exists:
  ```bash
  echo '{"required_linear_history":true,"allow_force_pushes":false,"required_status_checks":null,"enforce_admins":false,"required_pull_request_reviews":null,"restrictions":null}' \
    | gh api -X PUT "repos/{owner}/{repo}/branches/development/protection" --input -
  ```
  if it fails (plan limits on private repos, insufficient token scopes) → warn the user it must be set by hand, record nothing, continue — until then the flow's own discipline (no local trunk write except the FF land) is the only guard, surfaced, never silent
- create missing folders — `.sprint` / `.adr` / `.stories`, + `.brief` when none exists

### 1.0.3 Enforce the `.sprint` lifecycle

Read the existing plans (`.sprint/sprint-v{N}.md` — `status` frontmatter), then decide, **in this order**:

1. **A surviving draft** — a `draft` whose sprint is genuinely unfinished → **resume it**: keep its version number, no new version, no cascade — skip straight onward.
2. **Explicit override only** — the user explicitly asks to drop a surviving draft → warn first, then flip it to `abandoned` and fall through to 3. Never abandon on your own initiative.
3. **Otherwise** — set the next `v{N}`; **cascade** — archive everything older on the new draft; **backstop** — a previous draft whose PRs all landed but still reads `draft` is an irregular close (the flip normally rode its own sprint's spec-sweep commit, Refine 5.3.1) → forge-verify via `gh`: every PR with head `feat/sprint-v{M}` or `feat/sprint-v{M}-*` (divided chains) is closed *because landed* — its commits reachable from `origin/development`; an abandoned PR also reads closed and doesn't count → flip `draft → built` here.

Hold the one-draft rule — at most one `status: draft` in `.sprint/` at any time. Status edits are **annotations, not changes** — they ride 1.3.2's `docs(plan)` bundle, never their own commit. Decide the flips here; **1.0.5's tail applies them** after the branch is cut, so the edits are made on the sprint branch they belong to.

### 1.0.4 Fetch the trunk

`git fetch origin development` — refresh the trunk ref; no checkout. if no origin → skip.

### 1.0.5 Cut the sprint branch

Cut the branch off the fetched trunk, before any authoring:

```bash
git checkout -b feat/sprint-v{N} origin/development
```

Ungated (the gate is plan approval, 1.3.1), **local-only** — deterministic name, no remote ref until publication (Build 3.4.1). if no origin → `git checkout -b feat/sprint-v{N} development` (the local trunk seeded at 1.0.2). if `git status --porcelain` shows uncommitted state this session did not create → surface it to the user before switching — never blind-switch over another session's work (your own in-flight draft edits ride the checkout; that is the resume case, fine).

**Resume** (a surviving draft, 1.0.3): the branch `feat/sprint-v{N}` already exists → `git checkout feat/sprint-v{N}` — never cut a second branch for the same sprint. A draft written but never committed (the session died between 1.2.6 and 1.3.2) simply rides the working tree into this checkout; 1.3.2 commits it as the one `docs(plan)`.

Tail — after the cut (whichever branch above ran): **write the status flips 1.0.3 decided** (cascade archives · backstop `built` · any `abandoned`) — applied here, on the sprint branch, so they ride 1.3.2's `docs(plan)` bundle.

---

## 1.1 Discovery

**Goal:** understand the intent and map the touched code.

### 1.1.1 Discuss with user

Open context-aware — three openings, same interview style for all:

1. greenfield project — what are we building, from zero?
2. brownfield project — first contact with an existing codebase
3. new sprint — artifacts exist; what's this sprint for?

Then intent-driven — ask the intent, infer + confirm the rest (not a questionnaire):

1. **Intent** — what's this sprint for?
2. → `.stories` or architecture — user-facing want vs technical work
3. → `.adr` — decisions the approach forces
4. → `.brief` check — still fits north-star / non-goals? (mostly silent)

Interview style: one question at a time, each with a recommended answer; explore the codebase instead of asking where it can answer (quick inline lookups to steer the conversation — the big sweep is 1.1.2); challenge + sharpen the user's terms.

### 1.1.2 Codebase exploration

Once the intent is clear, the big sweep: dispatch `codebase-explorer` agents (parallel, `codebase-explorer-prompt`) to map only the touched modules — grounds the discussed intent in the actual code. *Direction from the session, discretion to the agent*: seed each with what the session already holds — the discussed intent, the modules to map, the `.spec` slice if present — then latitude: each expands its exploration to what its task needs, no rigid procedure.

if `.spec` exists → explorers read its slice first — the spec scopes + frames the sweep, the code carries the detail. if greenfield (no code yet) → skip.

### 1.1.3 Gate — confirm discovery

Present the discovery summary — sized to the sprint, no fixed length: the intent as understood, the touched-code map, anything 1.1.1 sharpened. The user confirms it captures their intent → proceed to Author; corrections → incorporate, re-present.

---

## 1.2 Author

**Goal:** decide the approach, then write the sprint's artifacts.

### 1.2.1 Create / edit `.brief`

if no `.brief` yet (greenfield, or brownfield first contact) → write the founding `.brief` per `brief-format` — the charter + the DoD home. Otherwise edit **only** when Discovery's `.brief`-check (1.1.1) surfaced a project-level discrepancy or add-on — scarce by design, the slowest-changing artifact.

### 1.2.2 Create / edit `.stories`

Add/edit `US-###` per `stories-format`. if a new want conflicts with an existing story → resolve it and route the WHY to `.adr` (1.2.4) — the decision log holds the reasoning, not the stories file.

### 1.2.3 Discuss the architecture

Discuss the approach with the user — how the sprint fits the existing architecture, what's new, which patterns hold, what the alternatives are. Let the discussion run naturally; note decisions as they emerge — formal capture is 1.2.4, don't interrupt the flow with paperwork.

### 1.2.4 Capture `.adr`

Capture the decisions in `.adr` per `adr-format` — **append-only**: never edit or delete an existing record. The bar is **judgment**, guided by Pocock's three tests — hard to reverse · surprising without context · a real trade-off — not a hard all-three checklist; when in doubt, the reasoning can stay in the plan. Check each new decision once against `.brief` principles (north-star / non-goals): deviation → flag to the user, never block. Downstream trusts the record (*trust the artifact*) — write it to be consumed.

### 1.2.5 Gate — approve `.adr`

The user confirms the written records say what 1.2.3 decided — **capture fidelity, not re-deciding**; mismatch → fix in 1.2.4, re-present. The commit gate stays 1.3.1.

### 1.2.6 Write the `.sprint` plan

Write `.sprint/sprint-v{N}.md` per `sprint-format` — the sprint's single pass/fail **Goal** + scope + epics + the `US-###` slice + success metric + DoD-ref; `status: draft`. This is the exact contract Tickets 2.0.2 reads. References, never copies — the `US-###` slice points at `.stories`, the DoD-ref at `.brief`.

---

## 1.3 Phase close

**Goal:** persist the work + hand off.

### 1.3.1 Gate — approve the `.sprint` plan

The user accepts the sprint — approves the plan's **Goal + scope**. The gate sits on content acceptance, never on a vcs op: once accepted, 1.3.2 runs autonomously — no "shall I commit?" follow-up. Edits → incorporate, re-present.

### 1.3.2 Finish documentation artifacts

Author **one** atomic commit of the documentation artifacts.

- **Greenfield first** — the founding charter is its own change, committed before the plan bundle, the trailer inside the quoted message (the audit contract holds from the branch's first commit):
  ```bash
  git add .brief
  git commit -m "docs(brief): <project> founding charter

  Assisted-by: session <model>"
  ```
- **The bundle** — the `.sprint` plan + any `.stories`/`.adr` edits + the 1.0.3 lifecycle flips: one logical change — *plan the sprint*, portfolio management included:
  ```bash
  git add .sprint .stories .adr
  git commit -m "docs(plan): sprint-v{N}

  Story: US-0XX, US-0YY
  ADR: ADR-0ZZ
  Assisted-by: session <model>"
  ```
  Message shape per `commit-format`. Trailers carry the captured `Story:`/`ADR:` pointers by ID (never a copy) plus `Assisted-by:` authored explicitly. `docs(plan)`/`docs(brief)` are **session-authored** (no dispatched writer), so the role token is **`session`** and `<model>` is the session's own runtime model id — never config-appended.

Follows the 1.3.1 acceptance — the gate approved the content; the commit is autonomous.

- **Resume tail** — if resuming (1.0.5) and a `docs(plan)` v{N} commit already sits on this branch → **never author a second one**: stage the new edits and amend it, re-stating the updated trailers (`git add … && git commit --amend`) — safe because nothing is pushed until Build-close (3.4.1); the one-`docs(plan)`-per-sprint principle holds across resumes.

### 1.3.3 No push (local until Build-close)

Plan finishes locally only — **no push**. Durability is local (the commits carry it), and the branch has no remote ref yet — publication is Build-close (3.4.1).

### 1.3.4 Handoff

Summarise what was captured: the plan's Goal + version, `.stories` touched (IDs + one-liners), ADRs (titles), the Brief if founded, and where the branch sits (`git log --oneline origin/development..HEAD` inline is fine). Recommend running `/sprint-tickets` in a **fresh session** — clean context; the artifacts carry the intent (*phases don't share a session — artifacts are the bridge*). This phase ends here — never start ticketing.

---

## Key principles

- **Gates sit on content, ops run autonomously** — three gates, zero op confirmations.
- **The branch is private clay, local** — no remote ref, no push, until Build publishes it.
- **One `docs(plan)` commit** — lifecycle flips are annotations riding it; greenfield's `docs(brief)` commits first.
- **Quality is owned at creation** — downstream consumes these artifacts as-given, in a fresh session; write them for a context-less reader, each fact in one place, referenced not restated.
- **Thin orchestration** — explorers own the exploration, the user owns the decisions; the session steers, gates, and commits.
