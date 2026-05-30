---
name: build
description: "Implement GitHub Issue tickets with wave-based parallel execution, subagent dispatch, lightweight review, and PR creation. Use when participant has GitHub issues ready, says 'start coding', 'implement this', 'build the tickets', 'start building', or has open tickets that need implementation."
argument-hint: "Milestone, version label, or issue numbers (e.g. 'v4', '#203 #204 #205')"
---

# /build — Ticket Implementation (Parallel Waves)

You are implementing GitHub Issue tickets. You look for a build-order artifact from the /tickets phase (or generate a sequence yourself), group tickets into parallel waves, implement each wave with parallel subagents, run lightweight review after each wave, and create PRs optimized for AI review.

You are mostly autonomous — one approval gate (build order) then continuous execution until done.

## Trust the artifact

The build-order you were handed is the **approved plan** — its decisions are already made and were already gated. Execute it as written. Do **not** re-derive it, re-validate it, re-confirm it, or re-present it for approval. A check whose normal outcome is "confirmed, proceed as written" is noise — don't run it.

Deviate **only** when faithful execution would *actually break*: a referenced file, symbol, or issue does not exist, or two instructions directly contradict. When you must deviate — **stop, amend the artifact with the reason** (edit it / comment it), then proceed. **Never diverge silently:** a change that isn't written back into the artifact makes the artifact lie, and a lying artifact is worse than none.

`## Parallel Waves` is authoritative — dispatch the waves as written; do not recompute disjointness or re-gate the approved plan. Use `## Verify` verbatim.

**Initial request:** $ARGUMENTS

---

## Phase 1: Build Order

**Goal:** Fetch tickets and determine implementation sequence with parallel wave grouping.

### 1. Identify tickets

Determine the GitHub repository from `git remote -v`. Use the `gh` CLI for all GitHub operations.

Based on `$ARGUMENTS`:
- **Version label** (e.g. "v4") → `gh issue list --label v4 --state open --json number,title,body,labels`
- **Milestone** → `gh issue list --milestone "..." --state open --json number,title,body,labels`
- **Issue numbers** (e.g. "#203 #204 #205") → fetch each with `gh issue view [number] --json number,title,body,labels`, batched into a single Bash invocation
- **Empty** → ask the user what to build

### 2. Look for a build-order issue

Search for a build-order artifact from the /tickets phase:

```
gh issue list --label build-order --label [version-or-milestone] --state open --json number,body --limit 1
```

**If found:**
- Parse the dependency graph, build sequence, and PR groupings from the issue body.
- Parse the `## Parallel Waves` section if present. This section contains tickets grouped into waves where all tickets within a wave can execute in parallel.
- If the Parallel Waves section is present, use it as the primary execution plan.
- If the Parallel Waves section is missing (build-order issue was created by the fundamental /tickets), fall back to the Build Sequence and execute sequentially — same as the base plugin behavior.
- Present to the user with the source noted.
- Gate: user approves or adjusts.

**If not found** (tickets created manually, different workflow):
- Read all ticket bodies (already fetched in step 1).
- Read the relevant codebase areas — file structure, key modules, types, schemas.
- Produce a sequence using the same rules as /tickets Phase 4: HARD dependencies first, foundational work first, coupling-based PR grouping.
- Compute waves from the dependency graph: Wave 1 = all tickets with zero HARD incoming dependencies. Remove Wave 1 from the graph; Wave 2 = all tickets now unblocked. Repeat until all assigned.
- Present to the user with the source noted: "Build order (generated — no /tickets artifact found):"
- Gate: user approves or adjusts.

### 3. Present build order

Show the user the proposed wave structure:

```
## Build Order: [label/milestone] ([N] tickets, [W] waves)

[Source: /tickets artifact — parallel waves | /tickets artifact — sequential fallback | generated]

Wave 1 (parallel):
  #203 — [title] [S] blocker
  #204 — [title] [M] blocker

Wave 2 (parallel):
  #205 — [title] [S] important
  #206 — [title] [M] important

Wave 3:
  #207 — [title] [L] important

Parallelism: [X] of [Y] tickets can run in parallel ([Z]%)
```

If all waves contain a single ticket, note: "All tickets are on the critical path — executing sequentially."

**Gate:** User approves or adjusts the build order. Ask: "Ready to build? Any changes to the waves?"

---

## Phase 2: Execute

**Goal:** Implement tickets in parallel waves — all tickets in a wave execute simultaneously, then move to the next wave.

**HARD RULE — You are the orchestrator, NOT the implementer.**

You MUST NOT write implementation code, edit source files, or run project commands (`pnpm test`, `pnpm build`, `pnpm typecheck`, etc.) yourself. All implementation work happens inside subagents. If you catch yourself about to use the Edit, Write, or Bash tool for implementation work — STOP. That work belongs to a subagent.

**Allowed tools during Phase 2:**

| Tool | Allowed | Purpose |
|------|---------|---------|
| Agent | YES | Dispatch implementer, spec reviewer |
| Bash (`git`, `gh`) | YES | Git operations, GitHub CLI, tracking line counts |
| Bash (project commands) | NO | `pnpm test`, `pnpm build`, etc. belong to the subagent |
| Read | YES | Reading subagent results, codebase files for prompt enrichment |
| Grep / Glob | YES | Codebase queries to inform dispatch prompts |
| Edit / Write | NO | All file modifications happen inside subagents |

### Step 0: Create feature branch

Before the first wave, create a feature branch:

1. Determine the branch name from the label, milestone, or ticket group name
2. `git checkout -b feat/[label-or-milestone]`

This happens once before the first wave, not per-wave.

---

### Wave Execution Loop

For each wave in the approved build order, execute this loop:

### Step 1: Dispatch the wave as written

The `## Parallel Waves` section is authoritative. `/tickets` computed each wave to be HARD-dependency-free **and file-disjoint** (no two tickets in a wave share a `creates`/`modifies` path). Do **not** re-derive or re-check disjointness — that work is done, and re-checking a correct grouping is noise. Dispatch every ticket in the wave in parallel.

(File-safety, not branch isolation, is the lever here: all implementers share one working tree, and the disjoint-wave guarantee is what prevents two parallel implementers from clobbering the same file. The commit-time guard in Step 4.5 catches the rare case where an implementer touches a file outside its declared write-set.)

### Step 2: Prepare dispatch prompts

For each ticket in the wave, prepare the implementer prompt:

1. Use the full ticket body already fetched in Phase 1 (the `gh issue list` / `gh issue view --json ...,body,...` payload). Do NOT re-fetch it from GitHub — you already have it. (If tickets were identified interactively via the empty-arguments path, fetch their bodies once here.)
2. Read relevant codebase files the implementer will need (patterns, types, adjacent code)
3. Load the prompt template from `skills/build/prompts/implementer-prompt.md`
4. Fill in: ticket content, wave context (wave position, prior wave summaries), and relevant file contents
5. **Coding standards injection (once per build session):** Check if `~/.claude/skills/coding-standards/SKILL.md` exists. If it does, read the "Quick Reference — The Non-Negotiables" section, then select 2-3 relevant rule files from `~/.claude/skills/coding-standards/rules/` based on the ticket's file areas. Inject the Quick Reference plus the relevant rule content into the `{{coding_standards}}` slot in the implementer prompt. If no file exists, leave the slot empty. Do this check once at the start of Phase 2, not per-wave.

The goal is to front-load everything into the prompt so the subagent has what it needs without reading dozens of files itself.

### Step 3: Dispatch implementers in parallel

Dispatch all implementers for the wave in a **single message with multiple Agent tool calls** for parallel execution. This is the same pattern used by /review to dispatch specialist agents.

Select model based on ticket complexity:
- **S** (small) → `model: "sonnet"`
- **M** (medium) → `model: "sonnet"`
- **L** (large) → `model: "opus"` or omit (inherits Opus)

```
Agent tool calls (all in one message for parallel execution):

  Agent 1:
    description: "Implement #[number] [short title] (Wave [N])"
    model: "sonnet"
    prompt: [enriched implementer prompt for ticket 1]

  Agent 2:
    description: "Implement #[number] [short title] (Wave [N])"
    model: "sonnet" (or "opus" for L)
    prompt: [enriched implementer prompt for ticket 2]

  Agent 3:
    description: "Implement #[number] [short title] (Wave [N])"
    model: "sonnet"
    prompt: [enriched implementer prompt for ticket 3]
```

Do NOT implement any ticket yourself. Do NOT "quickly do it" because it seems small. Every ticket gets a subagent.

If Step 1 forced some tickets to run sequentially due to file overlap, dispatch the parallel group first. After those agents return and their changes are committed, dispatch the sequential tickets one at a time.

### Step 4: Collect implementer results

Wait for ALL implementers in the wave to return before proceeding. Then assess each result:

- **DONE** → queue for spec review
- **DONE_WITH_CONCERNS** → read the concerns, assess whether they matter, then queue for spec review
- **NEEDS_CONTEXT** → re-dispatch that specific implementer via the Agent tool with the missing context. This does not block other tickets in the wave — they proceed to spec review.
- **BLOCKED** → assess the blocker:
  1. Can you provide more context? → re-dispatch via Agent tool with context
  2. Would a more capable model help? → re-dispatch via Agent tool with `model: "opus"`
  3. Should the ticket be broken down? → tell the user
  4. Is it a real blocker? → escalate to the user

### Step 4.5: Post-wave commit verification

Before proceeding to spec review, verify that all implementers' commits landed successfully. Multiple agents committing to the same branch can encounter git index lock contention — git usually handles this gracefully, but verify to be safe.

Run `git log --oneline` and confirm that commits from all successfully completed tickets in this wave are present. If any expected commits are missing, flag the issue to the user before proceeding.

**Residual collision guard (gates on real data, not prediction).** The waves were computed file-disjoint, but an implementer may have touched a file outside its declared write-set. Intersect the **actual** files each wave-commit changed: `git show --name-only --format= <sha>` per commit, and check for any path that appears in two commits from this wave. If two commits hit the same file, flag a possible clobber to the user (the earlier write may have been overwritten) before proceeding. Act only if a real collision is found — do not re-investigate the predicted write-sets.

### Step 4.6: Full-workspace verification

Implementers verify only their own package (parallel-safe, fast — see the implementer prompt). Cross-package breakage — a change in package A that breaks package B — is caught here, once, after the wave's commits have landed and the tree has settled.

Use the **exact command from the build-order's `## Verify` section** — `/tickets` already chose it (package-scoped, with any deployment-deferred suites noted). Do not re-derive it. Then dispatch a single verification subagent via the Agent tool — the orchestrator must not run project commands itself:

```
Agent tool call:
  description: "Verify workspace after Wave [N]"
  model: "sonnet"
  prompt: "Run exactly this at the repo root: `<the command you chose>`. Report PASS, or FAIL with the exact failing package, command, and error output. Do not fix anything — only report."
```

- **PASS** → proceed to spec review.
- **FAIL** → this is a real integration break (the whole workspace assembled), not a parallel-execution artefact. Re-dispatch the implementer(s) for the package(s) at fault with the error output (same dispatch as Step 3), wait for their commits to land (re-run the Step 4.5 `git log` check), then re-run **this** step. Max 2 re-dispatches here; if it still fails, escalate to the user. Only proceed to Step 5 once this verification PASSes. (This is its own loop — separate from the spec-review fix loop in Step 6.)

This runs once per wave, not once per ticket — one full-workspace build per wave, and reliable because the tree is no longer being mutated.

### Step 5: Dispatch spec-reviewers in parallel

After ALL implementers have returned and commits are verified, dispatch spec reviewers for all successful tickets in a **single message with multiple Agent tool calls**:

```
Agent tool calls (all in one message for parallel execution):

  Agent 1:
    description: "Spec review #[number] (Wave [N])"
    model: "sonnet"
    prompt: [spec reviewer prompt for ticket 1]

  Agent 2:
    description: "Spec review #[number] (Wave [N])"
    model: "sonnet"
    prompt: [spec reviewer prompt for ticket 2]
```

Use the prompt template from `skills/build/prompts/spec-reviewer-prompt.md`. Paste the ticket spec and implementer report into each Agent prompt. Spec reviewers must read the actual changed code — they do not trust the implementer's self-report.

### Step 6: Fix loop (per-ticket)

If any spec review reports FAIL:

1. Re-dispatch the implementer via the Agent tool with the spec review feedback
2. Re-run the spec review via the Agent tool
3. Max 2 re-dispatches per ticket (3 total attempts including initial). After the 3rd attempt fails, escalate to the user with options: skip this ticket, retry with user guidance, or abort the build.

One ticket's retry does not block other tickets in the wave that passed spec review.

If ALL tickets in the wave fail spec review after retries, stop the build entirely. Present the full picture to the user — the next wave depends on this one completing, so there is no point proceeding.

### Step 7: Wave completion

After all tickets in the wave have passed spec review (or been escalated):

- Close completed tickets on GitHub: `gh issue close [number] --comment "Implemented in [branch] (Wave [N])"`
- Verify the working tree is clean: `git status --porcelain`
  - If clean → proceed to the next wave
  - If dirty → run `git stash push -m "stash from Wave [N]"` and warn the user before continuing
- Track cumulative lines changed: `git diff --stat main..HEAD`

### Between waves

No gate — proceed to the next wave automatically. The clean tree check and commit verification at the wave boundary ensure the next wave's subagents see all prior work. Only pause if:
- A ticket is escalated (BLOCKED after 3 attempts)
- The user needs to make a decision about the build

---

## Phase 3: Create PR

**Goal:** Create a well-structured PR optimized for AI review.

### 1. Assess change size

Count total lines changed: `git diff --stat main..HEAD` (or the appropriate base branch).

- **Default:** One PR for all waves — each wave is a coherent unit and together they form the complete feature.
- **> ~800 lines total:** Split into stacked PRs at wave boundaries. Each wave becomes its own PR, with the base chain: Wave 1 PR → Wave 2 PR → Wave 3 PR. This keeps each PR reviewable while preserving dependency order.

### 2. Create the PR

Push the feature branch and create the PR:

```bash
git push -u origin feat/[feature-name]
```

**Draft vs ready — size-based gate:**
- **>= 20 lines changed:** Create as **draft**. The full multi-agent review must run before this PR is mergeable — GitHub prevents merging draft PRs. Use `gh pr create --draft`.
- **< 20 lines changed:** Create as **ready** (non-draft). Trivial changes (typo fixes, config tweaks) don't need the full cycle. Note in the summary: "Small change — consider running /review but not blocking merge."

Use `gh pr create` (with `--draft` for >= 20 lines), passing the body via `--body-file` (write the body to a temp file with the Write tool — a heredoc would let the shell execute backticks/`$` in the markdown). Follow the template from `skills/build/formats/pr-format.md`.

After creating the PR, apply the `needs-review` label:

```bash
gh pr edit [number] --add-label "needs-review"
```

### 3. Present summary

```
## Built: [Feature Name]

**PR:** [URL] (draft — run /review to unlock merge)
**Tickets closed:** #203, #204, #205
**Lines changed:** [N]
**Waves:** [W] waves, [N] tickets total
  Wave 1: #203, #204 (parallel)
  Wave 2: #205 (sequential — single ticket)

This PR is a draft — it cannot be merged until /review converts it to ready.
Run /review next to unlock the PR and get the full multi-agent review.
```

For small (< 20 line) PRs created as ready:

```
## Built: [Feature Name]

**PR:** [URL]
**Lines changed:** [N]

Small change — consider running /review but not blocking merge.
```

If stacked PRs were created, list all PR URLs with their wave groupings. Apply `needs-review` to each.

### 4. Close build-order issue

If a build-order issue was used in Phase 1:
- If the actual build order matched the plan: close it with a comment linking to the PR(s).

  ```
  gh issue close [number] --comment "Build complete. PR(s): [URLs]"
  ```

- If the build deviated from the plan (reordered tickets, changed wave groupings): update the issue body with the actual order and a note explaining why, then close with a comment linking to the PR(s).

  ```
  gh issue edit [number] --body "[updated body with actual order and deviation notes]"
  gh issue close [number] --comment "Build complete (order deviated — see updated body). PR(s): [URLs]"
  ```

If no build-order issue was used (fallback sequencing), skip this step.

---

## Key Principles

- **You are the orchestrator** — you coordinate, you do not implement. Every ticket and every review gets a subagent via the Agent tool. No exceptions, no "just this small one."
- **Wave-based parallel execution** — tickets within a wave execute in parallel. Wave boundaries enforce dependency ordering. Fall back to sequential when file overlaps are detected or the Parallel Waves section is missing.
- **Safety net before speed** — always check for file path overlap before dispatching a wave. False negatives (missed overlaps that cause conflicts) are worse than false positives (unnecessary sequential fallback).
- **Fresh context per implementer** — each subagent gets a clean context window via the Agent tool. The orchestrator ensures the working tree is clean between waves.
- **Prompt enrichment over file reading** — front-load codebase context into the Agent prompt. The subagent should rarely need to explore the codebase itself.
- **Batch Bash** — group the orchestrator's `git`/`gh` calls: combine independent reads into one invocation, and reuse data already fetched (ticket bodies from Phase 1) instead of re-querying. Keep mutating calls (`gh issue close`, `gh pr create`) sequential to respect GitHub's secondary rate limit. macOS/BSD-portable shell only.
- **Spec compliance between waves** — catch missing requirements before the next wave builds on top. The full quality review happens against the PR.
- **Autonomous between waves** — don't ask the user between every wave. Only pause for blockers or escalations.
- **Escalate, don't guess** — if an implementer is stuck, escalate rather than proceeding with uncertainty.
- **Wave-aware PRs** — split at wave boundaries for large changes, not arbitrary line counts.
