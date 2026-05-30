---
name: tickets
description: "Turn an approved PRD into AI-ready GitHub Issues with implementation detail and parallel wave analysis. Use when participant has an approved PRD, says 'break this down', 'create issues', 'make tickets', 'turn this into tasks', or has a completed PRD that needs implementation planning."
argument-hint: "Path to PRD (optional — auto-detects most recent)"
---

# /tickets — PRD to GitHub Issues

You are turning an approved PRD into actionable, AI-ready GitHub Issues. You work through six phases: prerequisites, codebase re-exploration, architecture & decomposition, issue creation, build ordering with wave analysis, and PRD status update. You are mostly autonomous — one approval gate before creating issues.

## Trust the artifact

The PRD and ADRs you were handed are the **approved plan** — its decisions are already made and were already gated. Execute it as written. Do **not** re-derive it, re-validate it, re-confirm it, or re-present it for approval. A check whose normal outcome is "confirmed, proceed as written" is noise — don't run it.

Deviate **only** when faithful execution would *actually break*: a referenced file, symbol, or issue does not exist, or two instructions directly contradict. When you must deviate — **stop, amend the artifact with the reason** (edit it / comment it), then proceed. **Never diverge silently:** a change that isn't written back into the artifact makes the artifact lie, and a lying artifact is worse than none.

The PRD's *what* is settled — your job is the *how*. Fresh codebase exploration is expected; re-opening the plan's decisions is not.

**Initial request:** $ARGUMENTS

---

## Phase 0: Prerequisites

**Goal:** Find and confirm the PRD, ensure it's committed.

1. **Find the PRD:**
   - If `$ARGUMENTS` contains a path, use that.
   - Otherwise, check `.prd/` for the most recently modified `.md` file.
   - Confirm with the user: "I found `.prd/prd-v1.md` — is this the one?"

2. **Check git status:**
   - Is the PRD committed? Run `git status` to check.
   - If uncommitted: commit it. PRDs need version history before tickets reference them.
     ```
     git add .prd/prd-v{N}.md
     git commit -m "docs: add PRD for {feature name}"
     ```

3. **Check GitHub remote:**
   - Run `git remote get-url origin` to check if a GitHub remote exists.
   - If no remote:
     - Ask the participant for a repo name (suggest the project folder name).
     - Create the repo: `gh repo create {name} --private --source=. --push`
     - Tell the participant: "I've created a GitHub repository for your project."
   - If remote exists, continue.

4. **Read and parse the PRD.** Identify:
   - Core features/epics to decompose
   - Architecture decisions already made
   - Scope boundaries (what's in, what's out)
   - Success metrics (tickets must cover these)

5. **Read ADRs (if they exist):**
   - Check for `.adr/ADR.md`. If present, read it.
   - These are architectural constraints from `/plan` — the reasoning behind decisions in the PRD.
   - Pass them to `code-architect` agents in Phase 2 so implementation designs respect the "why" behind decisions, not just the "what."
   - If no `.adr/` directory exists, skip silently.

---

## Phase 1: Codebase Re-exploration

**Goal:** Get fresh codebase context. The PRD may have been written days ago — the code may have changed.

**No gate — this phase is autonomous.**

1. Launch 2-3 `codebase-explorer` agents (sonnet, parallel). Use the prompt template from `skills/tickets/prompts/codebase-explorer-prompt.md` — focus agents on areas the PRD touches.

2. Read only the files where you need more than the explorers already surfaced — the explorers quote the relevant code, so don't re-read wholesale.
3. Compare exploration findings against the PRD's architecture section:
   - **Consistent** → proceed silently.
   - **Contradiction** → flag to user. The PRD has priority unless the code reveals an anti-pattern the PRD didn't account for.

---

## Phase 2: Architecture & Decomposition

**Goal:** Design the implementation and break it into right-sized tickets.

1. Launch `code-architect` agents (inherit, parallel). Each agent takes a different epic/feature from the PRD. Use the prompt template from `skills/tickets/prompts/code-architect-prompt.md`.

2. Read the agents' findings. Assemble the full breakdown.

3. **Decide structure based on size:**
   - **< 8 tickets** → flat structure. All issues at the same level, labels differentiate.
   - **8+ tickets** → hierarchical. Epic issues as parents, feature issues group tasks, task issues are atomic.

4. **Present the breakdown** to the user in a grouped format:

   ```
   ## Breakdown: [Feature Name] ([N] tickets)

   ### Epic: [Name] — [priority]

   **Feature: [Name]** — [priority]
   - [Ticket title] [complexity] — [priority]
   - [Ticket title] [complexity] — [priority]

   **Feature: [Name]** — [priority]
   - [Ticket title] [complexity] — [priority]
   ```

   Include key details: what each ticket covers, dependencies, and anything the user should know.

**Gate:**
The user must approve the breakdown before you create issues. Ask: "Ready to create these as GitHub Issues? Any changes first?"

---

## Phase 3: Create GitHub Issues

**Goal:** Create well-structured GitHub Issues with AI-ready content.

### Determine the repository

Check `git remote -v` to get the GitHub repository. Use the `gh` CLI for all GitHub operations.

### Create the labels this cycle introduces

The standing labels already exist on the repo and don't change between cycles — priority (`blocker`/`important`/`nice-to-have`/`low`), complexity (`S`/`M`/`L`), type (`bug`/`refactor`/`docs`), `build-order`, and the cycle-enforcement labels (`needs-review`/`needs-refine`/`cycle-complete`). Don't re-create or even list them: it's a round-trip per label that changes nothing on an established repo.

The only labels a cycle introduces are the hierarchy labels `epic:{name}` / `feature:{name}` and the version label `v{N}` when the version is new. Create just those, with a curated colour and description so they match the existing set (`--force` keeps a re-run idempotent):

```bash
gh label create "epic:checkout" --color 5319E7 --description "Epic: checkout flow" --force
gh label create "feature:cart"  --color FEF2C0 --description "Feature: cart"        --force
gh label create "v8"            --color 0E8A16 --description "V8 — Checkout & Cart" --force
```

Use the real epic/feature names and version from your breakdown; drop the `v{N}` line if that version label already exists.

### Create issues

For each ticket in the approved breakdown:

**Issue body format:** Use the template from `skills/tickets/formats/ticket-format.md`.

**Issue creation order:**
1. Create milestone if the PRD warrants one.
2. Create epic issues first (if hierarchical).
3. Create feature issues, referencing epic.
4. Create task issues, referencing feature and adding dependency links.
5. Apply labels: priority + version + complexity (+ hierarchy labels if applicable).

**One issue per `gh issue create` call — never a generated script.** Each create is its own Bash call. Read the new issue number from its output and use it in the next call. This keeps writes sequential (concurrent creates trip GitHub's secondary rate limit), keeps each create legible, and makes failures safe — if #9 fails, #1–8 already exist and you see exactly where it stopped.

**Pass the body with `--body-file`, not a heredoc.** Issue bodies are markdown full of backticks, `$`, and `()` — characters the shell executes inside a heredoc, which silently mangles the body or breaks the command. Write each body to a temp file with the Write tool, then point `gh` at it:

```bash
gh issue create --title "Add login form" --body-file /tmp/ticket-login.md --label "feature:login,v1,M"
# prints the new issue URL, e.g. .../issues/42 — note the 42 for the next body
```

**You thread the cross-references, not the shell.** Shell variables don't survive between separate Bash calls, so don't try to capture numbers into them. Instead: create in dependency order (parents before children), read each printed number, and write it literally into the next body file (`Part of #41`, `Blocked by #42`). If a create prints no number (gh failed or prompted), STOP and report — never create a child that references a parent that doesn't exist.

### Present summary

After all issues are created, present a grouped summary with URLs:

```
## Created: [Feature Name] ([N] tickets)

### Epic: [Name] (#[number])

**Feature: [Name] (#[number])** — [priority]
- #[number] [Title] [complexity] — [priority]
- #[number] [Title] [complexity] — [priority]

**Feature: [Name] (#[number])** — [priority]
- #[number] [Title] [complexity] — [priority]
```

---

## Phase 4: Build Order & Wave Analysis

**Goal:** Produce a build-order issue so /build knows the implementation sequence and which tickets can execute in parallel.

Produce a **self-contained** build-order issue that `/build` executes without ever re-reading the tickets. Use the exact structure in `${CLAUDE_PLUGIN_ROOT}/skills/tickets/formats/build-order-format.md`. Inputs: the code-architects' **write-sets** (`creates`/`modifies`) and **depends-on** (HARD/SOFT) fields, plus complexity and priority.

1. **Collect per-ticket data.** For each ticket record: complexity `[S/M/L]`, `depends-on` (HARD only constrains order; SOFT does not), and the **write-set** = `creates ∪ modifies` (exact file paths). A shared *directory* is not a conflict; a shared *file* is.

2. **Compute file-disjoint waves** (the load-bearing step — `/build` shares ONE working tree across all implementers, so two parallel tickets touching the same file is a silent clobber). Run the algorithm from `build-order-format.md`:
   - `ready` = tickets whose HARD `depends-on` are all in completed waves.
   - If `ready` is empty while tickets remain → HARD dependency **cycle**; flag to the user to reclassify one HARD→SOFT.
   - Fill the wave from `ready` in priority order (blocker > important > nice), adding a ticket **only if its write-set is disjoint from the files already used in this wave**. A ready ticket that shares a file with the wave waits for the next wave.
   - Repeat until all assigned. Each wave is therefore HARD-dep-free **and** file-disjoint.
   - **Accepted trade-off:** this can make waves narrower than HARD-dep-only grouping — correct, because determinism + no clobber beats peak parallelism under a shared tree.

3. **Group into PRs** by coupling (shared runtime boundary), not line count. Each PR independently reviewable.

4. **Decide scope.** State what to build as a **decision, not an option**: "Build all N. #X is stretch → build unless told otherwise." No open degrees of freedom for `/build` to ask about.

5. **Write the verify command.** The exact command `/build` runs at each wave boundary (package-scoped, e.g. `pnpm -F @pkg typecheck && pnpm -F @pkg test`); note any suites deferred to deployment.

6. **Create the build-order issue** following `build-order-format.md` exactly — header `Plan status: APPROVED — authoritative`, then `## Scope`, `## Verify`, `## Tickets` (with write-sets), `## Parallel Waves` (authoritative), `## Build Sequence` (sequential fallback for fundamental /build), `## PR Grouping`.
   - Title: `Build Order: [feature/version]`; Labels: `build-order`, version label.
   - Pin the issue: `gh issue pin [number]`.

No additional user gate — the user already approved the breakdown in Phase 2. The build order is a deterministic consequence of that breakdown.

**Dependency strength:**
- **HARD** — ticket B cannot compile/run without ticket A's output. Constrains wave order.
- **SOFT** — ticket B works without ticket A's output, but is better/cleaner with it. Does not constrain order.

---

## PRD Status Update

After all issues are created and the build-order issue is pinned, update the PRD frontmatter from `status: draft` to `status: built`. Read the PRD file, replace `status: draft` with `status: built` in the YAML frontmatter, and write it back.

---

## Key Principles

- **One ticket = one independently verifiable change** = roughly one PR.
- **AI-ready content** — explicit file paths, verifiable criteria, verification commands. No business justification (that's in the PRD).
- **No CLAUDE.md duplication** — tickets contain only the delta specific to this task.
- **Acceptance criteria are testable**, not subjective.
- **Dependencies are explicit** — blocked-by and blocks references using issue numbers.
- **Complexity is AI resource cost** (S/M/L), never time estimates.
- **Batch reads, sequence writes** — create only the new epic/feature/version labels (idempotent, no cross-references), but create issues one `gh issue create` call at a time so you can thread numbers between calls and fail safely; never bundle the creates into a script. Reuse the PRD/ADR content you already read instead of re-querying. macOS/BSD-portable shell only.
- **Wave analysis enables parallel execution** — tickets in the same wave have no HARD dependencies between them and can be implemented simultaneously by /build.
