---
name: sprint-tickets
description: "Turn an approved Sprint Plan into AI-ready GitHub Issues with implementation detail and parallel wave analysis. Use when user has an approved Sprint Plan, says 'break this down', 'create issues', 'make tickets', 'turn this into tasks', or has a completed Sprint Plan that needs implementation planning."
argument-hint: "Path to Sprint Plan (optional — auto-detects most recent)"
---

# /sprint-tickets — Sprint Plan to GitHub Issues

You are turning an approved Sprint Plan into actionable, AI-ready GitHub Issues. You work through six phases: prerequisites, codebase re-exploration, architecture & decomposition, issue creation, build ordering with wave analysis, and Sprint Plan status update. You are mostly autonomous — one approval gate before creating issues.

## Trust the artifact

The Sprint Plan and ADRs you were handed are the **approved plan** — its decisions are already made and were already gated. Execute it as written. Do **not** re-derive it, re-validate it, re-confirm it, or re-present it for approval. A check whose normal outcome is "confirmed, proceed as written" is noise — don't run it.

Deviate **only** when faithful execution would *actually break*: a referenced file, symbol, or issue does not exist, or two instructions directly contradict. When you must deviate — **stop, amend the artifact with the reason** (edit it / comment it), then proceed. **Never diverge silently:** a change that isn't written back into the artifact makes the artifact lie, and a lying artifact is worse than none.

The Sprint Plan's *what* is settled — your job is the *how*. Fresh codebase exploration is expected; re-opening the plan's decisions is not.

**Initial request:** $ARGUMENTS

---

## Phase 0: Prerequisites

**Goal:** Find and confirm the Sprint Plan, ensure it's committed.

1. **Find the Sprint Plan:**
   - If `$ARGUMENTS` contains a path, use that.
   - Otherwise, check `.sprint/` for the most recently modified `.md` file.
   - Confirm with the user: "I found `.sprint/sprint-v1.md` — is this the one?"

2. **Check git status:**
   - Is the Sprint Plan committed? Run `git status` to check.
   - If uncommitted: commit it. Sprint Plans need version history before tickets reference them.
     ```
     git add .sprint/sprint-v{N}.md
     git commit -m "docs: add Sprint Plan for {feature name}"
     ```

3. **Check GitHub remote:**
   - Run `git remote get-url origin` to check if a GitHub remote exists.
   - If no remote:
     - Ask the user for a repo name (suggest the project folder name).
     - Create the repo: `gh repo create {name} --private --source=. --push`
     - Tell the user: "I've created a GitHub repository for your project."
   - If remote exists, continue.

4. **Read and parse the Sprint Plan.** Identify:
   - Core features/epics to decompose
   - The user-stories slice — the `US-###` references this cycle delivers
   - The Architecture **shape-of-change pointer** (2-3 lines) — the full architecture block no longer lives here
   - Scope boundaries (the single in/out Scope table)
   - The Success-metric target (names the Brief north-star — tickets must cover the work behind it)
   - The Definition-of-Done **reference** to the Brief

5. **Read the upstream context artifacts (each skip-if-absent — greenfield/pre-migration safe):**
   - **Stories** — check for `.stories/STORIES.md`. If present, read it to resolve the `US-###` references in the Sprint Plan into their full `As a X, I want Y, so that Z` sentences. You thread these resolved sentences into the design inputs (Phase 2) and onto each ticket's traceability back-ref (Phase 3). If absent, skip silently.
   - **Brief** — check for `.brief/brief.md`. If present, read it for the canonical **Definition-of-Done** and the **Quality Goals** (3-5 attributes as guardrails). Tickets and their acceptance criteria must not contradict these. If absent, skip silently.
   - **ADRs** — check for `.adr/ADR.md`. If present, read it. These are architectural constraints from `/sprint-plan` — the reasoning ("why") behind decisions in the Sprint Plan. Treat each ADR's **Scope** as a **coarse subsystem boundary**, not a file-path/glob list — it tells you which subsystem a decision governs, not which files to touch. Pass the relevant ADRs (decision + Y-statement) to `code-architect` agents in Phase 2 so implementation designs respect the "why," not just the "what." If absent, skip silently.

---

## Phase 1: Codebase Context

**Goal:** Have accurate codebase context for the areas the Sprint Plan touches. How you get it depends on what's already available — use the cheapest tier that's sound. **No gate — this phase is autonomous.**

Pick the **first** tier that applies:

**Tier A — exploration already in context (combined plan→tickets session).** If `/sprint-plan` just ran in this same session, you already hold its Phase 2 codebase map (architecture, patterns, integration points) in this conversation. **Reuse it — do NOT dispatch explorer agents.** The code cannot have changed since planning seconds ago, so re-exploring is pure waste. Go straight to Phase 2 with the retained findings.

**Tier B — fresh session, `.spec/spec.md` exists.** Read the spec first — it IS the system map (maintained by `/sprint-refine`). Scope `codebase-explorer` agents (sonnet, parallel) to **only** the modules this Sprint Plan touches that the spec does not already describe; skip the Architecture-Mapping mode the spec already covers. If the spec was committed before recent code landed, re-explore any touched path with new commits since the spec's commit.

**Tier C — fresh session, no spec (first cycle).** Launch 2-3 `codebase-explorer` agents (sonnet, parallel) for the full 3-mode sweep. Use the prompt template from `skills/sprint-tickets/prompts/codebase-explorer-prompt.md` — focus agents on areas the Sprint Plan touches.

Then, regardless of tier:
- Read only the files where you need more than the explorers already surfaced — the explorers quote the relevant code, so don't re-read wholesale.
- Compare context against the Sprint Plan's Architecture **shape-of-change pointer** plus the Spec (`.spec/spec.md`) and the ADRs. The Sprint Plan no longer carries a full architecture block — its 2-3 line pointer says only "shape of the change," and the durable architecture lives in the Spec, the durable "why" in the ADRs. Reconcile against all three:
   - **Consistent** → proceed silently.
   - **Contradiction** → flag to user. The Sprint Plan + ADRs have priority unless the code reveals an anti-pattern they didn't account for.
- **Carve the Spec slices for Phase 2.** For each epic/feature, pull the matching `.spec/spec.md` sections (by their level-2 heading anchors) that describe the modules it touches. You forward this slice — not the whole spec — into each `code-architect` agent's design inputs, so the per-cycle design is grounded in the current system map.

---

## Phase 2: Architecture & Decomposition

**Goal:** Design the implementation and break it into right-sized tickets.

1. Launch `code-architect` agents (inherit, parallel). Each agent takes a different epic/feature from the Sprint Plan and **owns the per-cycle design for it** — the design lives in the agent's returned findings and flows straight into the tickets. **No design-doc file is produced or written anywhere.** Use the prompt template from `skills/sprint-tickets/prompts/code-architect-prompt.md`. Into each agent's design inputs, thread:
   - the resolved **US-### sentences** the epic/feature serves (from `.stories/STORIES.md`, if read in Phase 0) so the design stays anchored to the user want;
   - the relevant **Spec slice** you carved in Phase 1;
   - the governing **ADRs** — decision + **Y-statement** — so the design honours the recorded "why."

2. Read the agents' findings — this **is** the per-cycle design. Assemble the full breakdown directly from them; do not persist an intermediate design document.

3. **Decide structure based on size:**
   - **< 8 tickets** → flat structure. All issues at the same level, labels differentiate.
   - **8+ tickets** → hierarchical. Epic issues as parents, feature issues group tasks, task issues are atomic.

4. **Present the breakdown** to the user in a grouped format:

   ```
   ## Breakdown: [Feature Name] ([N] tickets)

   ### Epic: [Name]

   **Feature: [Name]**
   - [Ticket title] [complexity]
   - [Ticket title] [complexity]

   **Feature: [Name]**
   - [Ticket title] [complexity]
   ```

   Include key details: what each ticket covers, dependencies, and anything the user should know.

**Gate:**
The user must approve the breakdown before you create issues. Ask: "Ready to create these as GitHub Issues? Any changes first?"

---

## Phase 3: Create GitHub Issues

**Goal:** Create well-structured GitHub Issues with AI-ready content.

### Determine the repository

Check `git remote -v` to get the GitHub repository. Use the `gh` CLI for all GitHub operations.

### Ensure labels exist

Two kinds of label feed this cycle — the **standing set** (identical every cycle) and the **per-cycle set** (hierarchy + version). The full taxonomy with colours lives in `${CLAUDE_PLUGIN_ROOT}/skills/sprint-tickets/formats/labels.md` — the single source of truth.

**Standing set.** `type/*`, `complexity/*`, `build-order`, and the cycle-enforcement labels `needs-review`/`needs-refine`/`cycle-complete`. These drive the cycle state machine — a missing one silently fails the `--add-label` that flips a PR forward — so they must exist before any issue or PR is touched. **Do NOT assume they exist:** a fresh repo (including one this skill just created via `gh repo create`) has none of them. Run the **Ensure block** from `labels.md` once in a single Bash call — every command is `--force` (create-or-update), so on an established repo it's a no-op and on a fresh repo it seeds the whole machine. There is **no `priority/*` family** — wave-fill ties break on issue number, not priority.

**Per-cycle set.** The hierarchy labels `epic:{name}` / `feature:{name}` and the version label `v{N}` — **`v{N}` derives from the Sprint Plan filename** (`.sprint/sprint-v{N}.md` → `v{N}`). Create these from your breakdown (`--force` keeps a re-run idempotent):

```bash
gh label create "epic:checkout" --color 6F42C1 --description "Epic: checkout flow" --force
gh label create "feature:cart"  --color BFD4F2 --description "Feature: cart"        --force
gh label create "v8"            --color 0052CC --description "V8 — Checkout & Cart" --force
```

Use the real epic/feature names and version from your breakdown.

### Create issues

For each ticket in the approved breakdown:

**Issue body format:** Use the template from `skills/sprint-tickets/formats/ticket-format.md`. Every ticket body carries:
- the **US-### back-ref** — the user story this ticket serves (resolved from `.stories/STORIES.md` in Phase 0; required when stories exist);
- the **parent Sprint-Plan link** — `sprint-v{N}` (required);
- an **optional governing-ADR pointer** — only when an ADR constrains this change;
- **Complexity** (S/M/L) in the body — AI resource cost, never a time estimate;
- a one-line **Spec pointer** (`.spec/spec.md#anchor`) for current behavior.

**Issue creation order:**
1. Create milestone if the Sprint Plan warrants one.
2. Create epic issues first (if hierarchical).
3. Create feature issues, referencing epic.
4. Create task issues, referencing feature and adding dependency links.
5. Apply labels: version (`v{N}`, from the sprint-vN filename) + complexity (`complexity/{S,M,L}`) (+ hierarchy labels if applicable).

**One issue per `gh issue create` call — never a generated script.** Each create is its own Bash call. Read the new issue number from its output and use it in the next call. This keeps writes sequential (concurrent creates trip GitHub's secondary rate limit), keeps each create legible, and makes failures safe — if #9 fails, #1–8 already exist and you see exactly where it stopped.

**Pass the body with `--body-file`, not a heredoc.** Issue bodies are markdown full of backticks, `$`, and `()` — characters the shell executes inside a heredoc, which silently mangles the body or breaks the command. Write each body to a temp file with the Write tool, then point `gh` at it:

```bash
gh issue create --title "Add login form" --body-file /tmp/ticket-login.md --label "feature:login,v1,complexity/M"
# prints the new issue URL, e.g. .../issues/42 — note the 42 for the next body
```

**You thread the cross-references, not the shell.** Shell variables don't survive between separate Bash calls, so don't try to capture numbers into them. Instead: create in dependency order (parents before children), read each printed number, and write it literally into the next body file (`Part of #41`, `Blocked by #42`). If a create prints no number (gh failed or prompted), STOP and report — never create a child that references a parent that doesn't exist.

### Present summary

After all issues are created, present a grouped summary with URLs:

```
## Created: [Feature Name] ([N] tickets)

### Epic: [Name] (#[number])

**Feature: [Name] (#[number])**
- #[number] [Title] [complexity]
- #[number] [Title] [complexity]

**Feature: [Name] (#[number])**
- #[number] [Title] [complexity]
```

---

## Phase 4: Build Order & Wave Analysis

**Goal:** Produce a build-order issue so /sprint-build knows the implementation sequence and which tickets can execute in parallel.

Produce a **self-contained** build-order issue that `/sprint-build` executes without ever re-reading the tickets. Use the exact structure in `${CLAUDE_PLUGIN_ROOT}/skills/sprint-tickets/formats/build-order-format.md`. Inputs: the code-architects' **write-sets** (`creates`/`modifies`) and **depends-on** (HARD/SOFT) fields, plus complexity.

1. **Collect per-ticket data.** For each ticket record: complexity `[S/M/L]`, `depends-on` (HARD only constrains order; SOFT does not), and the **write-set** = `creates ∪ modifies` (exact file paths). A shared *directory* is not a conflict; a shared *file* is.

2. **Compute file-disjoint waves** (the load-bearing step — `/sprint-build` shares ONE working tree across all implementers, so two parallel tickets touching the same file is a silent clobber). Run the algorithm from `build-order-format.md`:
   - `ready` = tickets whose HARD `depends-on` are all in completed waves.
   - If `ready` is empty while tickets remain → HARD dependency **cycle**; flag to the user to reclassify one HARD→SOFT.
   - Fill the wave from `ready` in issue-number order (deterministic — there is no priority), adding a ticket **only if its write-set is disjoint from the files already used in this wave**. A ready ticket that shares a file with the wave waits for the next wave.
   - Repeat until all assigned. Each wave is therefore HARD-dep-free **and** file-disjoint.
   - **Accepted trade-off:** this can make waves narrower than HARD-dep-only grouping — correct, because determinism + no clobber beats peak parallelism under a shared tree.

3. **Group into PRs** by coupling (shared runtime boundary), not line count. Each PR independently reviewable.

4. **Decide scope.** State what to build as a **decision, not an option**: "Build all N. #X is stretch → build unless told otherwise." No open degrees of freedom for `/sprint-build` to ask about.

5. **Write the verify command.** The exact command `/sprint-build` runs at each wave boundary (package-scoped, e.g. `pnpm -F @pkg typecheck && pnpm -F @pkg test`); note any suites deferred to deployment.

6. **Create the build-order issue** following `build-order-format.md` exactly — header `Plan status: APPROVED — authoritative`, then `## Scope`, `## Verify`, `## Tickets` (with write-sets), `## Parallel Waves` (authoritative), `## Build Sequence` (sequential fallback for fundamental /build), `## PR Grouping`.
   - Title: `Build Order: [feature/version]`; Labels: `build-order`, version label.
   - Pin the issue: `gh issue pin [number]`.

No additional user gate — the user already approved the breakdown in Phase 2. The build order is a deterministic consequence of that breakdown.

**Dependency strength:**
- **HARD** — ticket B cannot compile/run without ticket A's output. Constrains wave order.
- **SOFT** — ticket B works without ticket A's output, but is better/cleaner with it. Does not constrain order.

---

## Sprint Plan status — do not flip here

`/sprint-tickets` does **not** change the Sprint Plan's `status:`. It stays `draft`: the cycle is still being built against, and `draft` covers planning *and* building-against (per `${CLAUDE_PLUGIN_ROOT}/skills/sprint-plan/formats/sprint-format.md`). `/sprint-build` is the skill that flips `draft → built`, and only once the code is actually done and the PR exists — so the status never claims "code is done" before it is. Leave `.sprint/sprint-v{N}.md` untouched in this skill.

---

## Key Principles

- **One ticket = one independently verifiable change** = roughly one PR.
- **US-### traceability is load-bearing** — every ticket carries its user-story back-ref (plus the parent `sprint-v{N}` link and, when one governs, an ADR pointer). This back-ref is what `/sprint-refine`'s derived **coverage view** reads to map built-vs-unbuilt stories — a ticket with no `US-###` is invisible to it.
- **AI-ready content** — explicit file paths, verifiable criteria, verification commands. No business justification (that's in the Sprint Plan).
- **No CLAUDE.md duplication** — tickets contain only the delta specific to this task.
- **Acceptance criteria are testable**, not subjective.
- **Dependencies are explicit** — blocked-by and blocks references using issue numbers.
- **Complexity is AI resource cost** (S/M/L), never time estimates.
- **Batch reads, sequence writes** — ensure the standing labels and create the per-cycle epic/feature/version labels up front (all `--force`, idempotent), but create issues one `gh issue create` call at a time so you can thread numbers between calls and fail safely; never bundle the creates into a script. Reuse the Sprint Plan/ADR content you already read instead of re-querying. macOS/BSD-portable shell only.
- **Wave analysis enables parallel execution** — tickets in the same wave have no HARD dependencies between them and can be implemented simultaneously by /sprint-build.
