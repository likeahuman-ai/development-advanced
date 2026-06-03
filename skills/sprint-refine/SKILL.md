---
name: sprint-refine
description: "Fix review findings and update the system spec (.spec/spec.md). Sprint-aware — auto-detects a stack of PRs and fixes the whole sprint on the tip in one pass. Use when user has review comments on a PR (or a stack of PRs from one sprint), says 'fix these', 'address the review', 'refine my code', 'refine the sprint', 'update the spec', 'sync the spec', 'close out this cycle', or wants to finalize work after /sprint-review."
argument-hint: "PR number or URL (optional — auto-detects current branch PR)"
---

# /sprint-refine — Fix Review Findings + Patch Spec

You are closing out a development cycle. Phase 1: fix review findings from the PR (blocked work is surfaced to you, and parked in the findings backlog if unresolved). Phase 2: patch the system spec by lightweight delta to reflect what exists now. Phase 3: run the Definition-of-Done gate, present the derived coverage view, and close the cycle.

You are mostly autonomous — three gates: the user picks which findings to fix, then approves the spec delta, then the close-out DoD gate must pass before the cycle completes.

## Trust the artifact

The review findings you were handed are the **approved plan** — its decisions are already made and were already gated. Execute it as written. Do **not** re-derive it, re-validate it, re-confirm it, or re-present it for approval. A check whose normal outcome is "confirmed, proceed as written" is noise — don't run it.

Deviate **only** when faithful execution would *actually break*: a referenced file, symbol, or issue does not exist, or two instructions directly contradict. When you must deviate — **stop, amend the artifact with the reason** (edit it / comment it), then proceed. **Never diverge silently:** a change that isn't written back into the artifact makes the artifact lie, and a lying artifact is worse than none.

Findings were already scored and filtered — fix them, don't re-adjudicate validity. (The user chooses *which* to fix; that's their gate, not your re-litigation.)

**Initial request:** $ARGUMENTS

---

## Phase 0: Scope — single PR or stacked sprint

A `/sprint-build` run over ~800 lines ships the cycle as **stacked PRs** — one per wave, base chain `Wave1 ← Wave2 ← … ← tip` (see `skills/sprint-build/formats/pr-format.md`). Running `/sprint-refine` on whichever branch you're standing on would fix that one PR's findings and then run the **sprint-level close-out** (spec patch, DoD gate, `cycle-complete`) while the *other waves' findings sit unaddressed* — the cycle gets marked done with real bugs still open. So establish scope first.

Find the anchor PR — the `$ARGUMENTS` PR if given, else the current branch's (`gh pr view --json number,headRefName,baseRefName`). Then pull the open PRs and walk the base→head chain:

```bash
gh pr list --state open --json number,title,headRefName,baseRefName,additions,deletions
```

The anchor belongs to a **stack** if its `baseRefName` is another open PR's `headRefName`, **or** another open PR's `baseRefName` is the anchor's `headRefName`. Follow both directions to collect the connected chain; order it base-first. The **tip** = the PR whose head is not any other PR's base; the **root base** = the chain root's `baseRefName` (normally `main`).

- **No chain (single PR)** → **single-PR mode**: Phases 1–3 below run exactly as written, for this one PR. The common case is untouched.
- **Chain of N** → **sprint mode**: the cycle *is* the whole stack. Follow the **Sprint mode** delta in each phase — pool every wave's findings and fix them on the tip (Phase 1), patch the spec once over the union diff (Phase 2), and run the close-out once for the whole sprint (Phase 3). Tell the user: *"This PR is wave k of a stack of N — refining the whole sprint: I'll fix every wave's findings on the tip, patch the spec once, and close the cycle when all N pass."*

  **Readiness check:** every PR in the chain must already carry a `### Code Review` comment (be `needs-refine`). If any wave is still `needs-review` (unreviewed), stop and tell the user to run `/sprint-review` first (it reviews the whole stack) — refining against a missing review would skip that wave's findings.

---

## Phase 1: Fix Review Findings

Read findings from GitHub, let user choose which to fix, dispatch implementer subagents, commit and push.

> **Sprint mode (stacked PRs) — pool findings, fix on the tip.** When Phase 0 found a stack, Phase 1 runs **once for the whole sprint**, not per PR. The tip already contains every wave's code (it sits atop the base chain), so all findings are addressable in one working tree. Steps 1–6 below change as follows:
>
> 1. **Gather every wave's findings** — read the `### Code Review` comment from **each** PR in the chain (batch the `gh pr view [n] --json comments` reads), parse each per `finding-format.md`, and **pool** them, tagging each finding with its source PR so attribution survives.
> 2. **Present ONE consolidated list** (not N), grouped by severity, each finding tagged with its wave (`#a, wave 1`). **Dedup across waves:** when the same finding-class recurs on multiple PRs, present it once and note where it recurs — it's one fix. Same gate: user picks `all / numbers / none`.
> 3. **Checkout the tip** so every fix lands in the cumulative tree: `gh pr checkout [tip]`.
> 4. **Group by `Files:` across the whole pool**, then dispatch — the existing same-file rule, applied sprint-wide. Build the groups so **every file is owned by exactly one implementer**: merge any findings that share a file into one group, and a multi-file finding pulls all *its* files into that same group (connected components of the finding↔file graph). So two findings touching the same file — even from different waves — land on one implementer and never clobber, and a finding spanning two files is fixed by a single implementer that owns both. (This is exactly why `Files:` must list every touched path.) Batch the reads, dispatch all groups in one parallel message.
> 5. **One commit on the tip:** `git add [fixed files]; git commit -m "fix: address review findings from sprint-v{N} (PRs #a–#z)"; git push`. Do **not** fix each wave on its own branch — the tip carries the whole stack to merge.
> 6. **Deferred findings → backlog, deduped across the sprint** — same as single-PR mode, but dedup by file+line **across all waves' findings**, so a bug recurring on three PRs is parked once, not three times.
>
> Then continue to Phase 2 (Sprint mode) and Phase 3 (Sprint mode). The per-PR steps below are the single-PR path.

### 1. Find the PR

- If `$ARGUMENTS` contains a PR number or URL, use that.
- Otherwise, detect current branch PR: `gh pr view --json number,title,url`
- If no PR: "No PR found. Specify a PR number or URL."

### 2. Check for review completion

Before reading findings, verify that `/sprint-review` has actually run on this PR. The review phase posts a comment starting with `### Code Review` and swaps the PR label from `needs-review` to `needs-refine`.

```bash
gh pr view [number] --json comments --jq '.comments[].body'
```

Parse for structured /sprint-review findings using the format in `${CLAUDE_PLUGIN_ROOT}/skills/sprint-refine/formats/finding-format.md`. Look for the most recent comment matching the format (starts with `### Code Review`).

**If no `### Code Review` comment exists on the PR:** This PR hasn't been through `/sprint-review` yet. Tell the user: "This PR hasn't been reviewed yet. Run `/sprint-review` first — it runs the specialist agents and posts findings to the PR. Then come back to `/sprint-refine` to fix them and update the spec." Do not proceed.

### 3. Handle edge cases

- **Review comment says "No issues found":** "Review was clean. Nothing to fix — skipping to spec update."
- **Comments exist but no structured findings:** "I found comments but they don't match the /sprint-review format. Want me to read them and address manually, or skip to spec update?"

If no findings to fix, skip directly to Phase 2 (spec update).

### 4. Present to user

```
## Review Findings: PR #[number]

### Critical
1. [description] — [file:line]

### Important
2. [description] — [file:line]

Which findings should I fix? (all / numbers / none)
```

**Gate:** User selects. "none" → skip to Phase 2.

### 5. Dispatch fixes (batched + parallel)

**You are the orchestrator, NOT the fixer.** Each fix gets a subagent.

First, **batch all the reads**: in a single message, read the code context (30-50 lines) for every selected finding at once. When several findings cluster in one file, read that file once rather than per-finding.

Then **dispatch all fix implementers in one parallel message** (sonnet) — the same shape `/sprint-build` uses for a wave — instead of read-then-dispatch one finding at a time. Each implementer gets the finding, its code context, and the suggested fix, via `${CLAUDE_PLUGIN_ROOT}/skills/sprint-refine/prompts/fix-prompt.md`.

Collect results: DONE → done; BLOCKED → report to user and continue with the rest. **File safety:** if two findings touch the same file, hand both to a single implementer so parallel fixes don't clobber each other.

**BLOCKED items — surface, don't auto-file.** Above-threshold findings are all meant to be fixed, so a `BLOCKED` is an exception: **report it to the user in this session** (the blocker reason) and continue with the rest. If the user can't resolve it now, **park it in the findings backlog** (`.sprint/backlog.md` — the same dump `/sprint-review` writes) so it isn't lost:

```markdown
## BLOCKED {one-line finding} — PR #[number], {date}
- File: {path}:{line}
- Why blocked: {blocked-reason | user-chose-to-defer}
```

Do **NOT** file GitHub Issues for these — Issues are committed work only; deferred signal lives in the findings backlog, which nothing reads automatically. Deduplicate against existing backlog entries by file+line.

### 6. Commit and push

```bash
git add [fixed files]
git commit -m "fix: address review findings from PR #[number]"
git push
```

---

## Phase 2: Patch the Spec (delta)

**Goal:** Bring `.spec/spec.md` into line with the system as it stands NOW. The spec is **cycle-living**: it is patched by a **lightweight delta**, never rewritten — the `spec-writer` agent emits and applies ADDED / MODIFIED / REMOVED hunks scoped to only the sections the diff touched. Git is the archive; there is no `changes/` or `archive/` tree.

**The spec sections** (stable level-2 heading anchors — never rename/reorder without updating inbound pointers): Architecture; Runtime / Data-flow view; Data Model; API Surface; Crosscutting Concepts & Patterns; Stack; Directory pointer-map; Infrastructure.

### Determine mode

- **No `.spec/` directory** → Create it: `mkdir -p .spec`
- **No `.spec/spec.md` exists** → Creation mode (first cycle) — the ONLY mode where the spec-writer writes free-hand prose.
- **`.spec/spec.md` exists** → Delta mode (subsequent cycle) — the spec-writer emits + applies delta hunks, not a rewrite.

### Creation mode (first cycle, no spec exists)

The system has never been documented. Full exploration needed.

1. **Explore the codebase.** Dispatch 2-3 `codebase-explorer` agents (sonnet, parallel) to map: architecture (components, connections); runtime / data-flow; data model (schemas, tables); API surface (endpoints, contracts); crosscutting patterns; the directory pointer-map; infrastructure / deploy target.

2. **Gather additional context (each read is skip-if-absent — greenfield/pre-migration safe):**
   - Read `.sprint/sprint-v{latest}.md` — what was built this cycle.
   - Read `.adr/ADR.md` — decisions that shape the design.
   - Read `package.json` — dependencies = stack.

3. **Dispatch spec-writer agent** (sonnet) with: explorer results, the Sprint Plan, the ADR content, and `${CLAUDE_PLUGIN_ROOT}/skills/sprint-refine/formats/spec-format.md` (template). Instruction: **creation mode** — fill all spec sections from the codebase context (types and lists over prose).

### Delta mode (subsequent cycles)

The spec exists. The PR diff scopes what changed; the spec-writer patches only those sections.

1. **Gather inputs (skip-if-absent where noted):**
   - Read `.spec/spec.md` — current baseline.
   - Get PR diff: `gh pr diff [number]`
   - Read full content of files touched by the diff (not just hunks).
   - Get directory listing: `ls src/` (drift check against the Directory pointer-map).
   - Read `package.json` (drift check against Stack).
   - Read `.adr/ADR.md` if it exists (new decisions this cycle — skip if absent).

   **Sprint mode — union diff (NOT `gh pr diff [tip]`):** for a stack, `gh pr diff [tip]` diffs the tip against its *parent wave* — it captures only the tip's slice and **silently drops the lower waves from the spec**. After `gh pr checkout [tip]`, compute the true union of the whole sprint instead:

   ```bash
   TIP=$(git rev-parse HEAD); BASE=$(git merge-base [root-base] $TIP); git diff $BASE..$TIP
   ```

   Feed that union diff — and the full content of every file `git diff --name-only $BASE..$TIP` lists — to the spec-writer in place of the single-PR diff. The spec-writer's section anchors are additive, so the union yields the same sections as N per-wave diffs would, minus the half-states. Same delta-mode contract; just the whole sprint's change, patched once.

2. **Dispatch spec-writer agent** (sonnet) with: the existing spec, the PR diff (single-PR mode) **or the union diff (sprint mode)**, the full touched files, the directory listing, `package.json`, any ADRs, and `${CLAUDE_PLUGIN_ROOT}/skills/sprint-refine/formats/spec-format.md` (template). Instruction: **delta mode** — emit ADDED / MODIFIED / REMOVED hunks naming the target section (and heading anchor) and **apply them**. Touch only changed sections; leave every unaffected section byte-for-byte alone. Add new entries, remove deleted ones, never re-describe untouched prose. Git is the archive — do not keep an in-file changelog.

### Review gate

After spec-writer returns:

1. Present the applied delta (the ADDED / MODIFIED / REMOVED hunks, or the diff against the previous version) to the user — not the whole file.
2. Ask: "Here's the spec delta for this cycle. Look good? (approve / edit)"
3. **Gate:** User must approve before commit.

If user wants edits: apply their changes, then proceed.

### Commit spec

```bash
git add .spec/spec.md
git commit -m "docs: patch spec after sprint-v{N} cycle"
git push
```

---

## Phase 3: Close Cycle

> **Sprint mode (stacked PRs) — close the whole sprint once.** The close-out is sprint-level: the DoD gate, coverage view, and `cycle-complete` apply to the *whole stack*, not each wave. Run Phase 3 **once**, only after every wave's findings are fixed (Phase 1) and the spec is patched (Phase 2). Deltas: the DoD gate holds the combined result on the tip against the Brief DoD + the single Sprint Goal (one Goal per sprint, so the gate is naturally sprint-level); the coverage view's **PR** column spans the stack (map each `US-###` to the wave/PR that delivered it); and the label flip applies to **every** PR in the chain. Don't flip any wave to `cycle-complete` until the gate passes for the whole stack — a sprint isn't done until all waves pass.

### 1. Definition-of-Done gate

Before marking the cycle complete, run the DoD gate. Gather (each read **skip-if-absent** — greenfield/pre-migration safe):

- The canonical **Definition-of-Done** from `.brief/brief.md` (the Brief is its only home). Skip silently if no Brief.
- The cycle **Goal** — the first line of `.sprint/sprint-v{latest}.md` (`This cycle succeeds iff <one objective>`). Skip silently if no Sprint Plan.

Hold the work against both: the Brief's DoD bar AND the single pass/fail cycle Goal. **Require a pass.** If either is unmet — name the unmet criterion, do NOT flip the label to `cycle-complete`, and tell the user what remains (e.g. a missed Goal objective, an unfixed finding, a DoD bar not cleared). The cycle does not close until the gate passes. (If both files are absent, there is nothing to gate against — note that and proceed.)

### 2. Coverage view

DERIVE and present the traceability for this cycle — there is **no matrix file** (built-vs-unbuilt is derived, never stored). Map each user story this cycle delivered through to its evidence:

`US-### → Issue → PR → spec`

- **US-###** — the story IDs the cycle's Sprint Plan referenced (read `.sprint/sprint-v{latest}.md`; resolve the sentences from `.stories/STORIES.md` if present — both skip-if-absent).
- **Issue** — the GitHub Issue(s) that implemented each story.
- **PR** — this PR (#[number]). **Sprint mode:** the wave/PR that delivered each story — a story may map to any wave in the stack (#a…#z), so the column spans the whole chain, not one PR.
- **spec** — the spec section/anchor (`.spec/spec.md#<anchor>`) the change is now documented under.

Render it as a short derived list (one row per US-###). This is a computed view, not a stored artifact — do not write it to a file.

### 3. Update cycle label

Mark the PR as cycle-complete — all phases have run AND the DoD gate passed:

```bash
gh pr edit [number] --remove-label "needs-refine" --add-label "cycle-complete"
```

**Sprint mode:** flip **every** PR in the chain, not just the anchor — one `gh pr edit [n] --remove-label "needs-refine" --add-label "cycle-complete"` per PR, kept sequential. The whole stack reaches `cycle-complete` together, because the DoD gate passed for the combined result on the tip. If any wave had an unfixable BLOCKED finding that you chose to leave open (not defer to an Issue), do **not** flip the stack — report it incomplete so the cycle stays open.

### 4. Present summary

```
## Cycle Complete: PR #[number]

**DoD gate:** passed (Goal met, Brief DoD cleared)
**Fixes:** [N] applied, [N] skipped, [N] blocked → [N] parked in the findings backlog
**Spec:** [created | patched] — .spec/spec.md
**Coverage:** US-### → Issue → PR → spec (derived above)

The full cycle ran: /sprint-plan → /sprint-tickets → /sprint-build → /sprint-review → /sprint-refine.
Next cycle: run `/sprint-plan` to start planning the next feature.
```

**Sprint mode** — present one summary for the whole stack instead of N:

```
## Cycle Complete: sprint-v[N] — stack of [N] PRs (#a–#z)

**DoD gate:** passed (Goal met, Brief DoD cleared — held against the combined result on the tip)
**Fixes:** [N] applied across [N] waves, [N] skipped, [N] blocked → [N] parked in the findings backlog (deduped across waves)
**Spec:** patched once over the sprint union diff — .spec/spec.md
**Coverage:** US-### → Issue → PR(wave) → spec (derived above)
**PRs:** #a … #z all flipped to cycle-complete — merge the stack base-first; the spec rides the tip.

The full cycle ran: /sprint-plan → /sprint-tickets → /sprint-build → /sprint-review → /sprint-refine.
Next cycle: run `/sprint-plan` to start planning the next feature.
```

---

## Key Principles

- **You are the orchestrator** — subagents fix code and write the spec. You coordinate.
- **Session-boundary safe** — reads everything from disk/GitHub. Works in a fresh session. Every new read of `.brief/`, `.stories/`, or `.sprint/` is **skip-if-absent** (greenfield/pre-migration safe).
- **Sprint-aware — fix the whole stack, close once** — a `/sprint-build` over ~800 lines ships a stack of PRs. Phase 0 detects it; in sprint mode `/sprint-refine` pools every wave's findings and fixes them on the **tip** (one working tree, grouped by `Files:` so no cross-wave clobber), patches the spec **once** over the sprint union diff (`merge-base..tip`, never `gh pr diff [tip]`), and runs the close-out **once** — flipping every wave to `cycle-complete` only when the DoD gate passes for the combined result. The single-PR path is unchanged; the stack branch engages only when the anchor PR is part of a chain.
- **Delta model, not rewrite** — the spec is cycle-living. The `spec-writer` emits + applies ADDED / MODIFIED / REMOVED hunks scoped to only the sections the diff touched; unaffected sections stay byte-for-byte. Git is the archive — no in-file changelog, no `changes/` tree. Free-hand rewrites are creation-mode only.
- **Three gates** — (1) **fix gate**: the user picks which findings to fix; (2) **spec gate**: the user approves the spec delta before commit; (3) **Definition-of-Done gate**: at close-out, the work must clear the Brief's canonical DoD AND the single pass/fail cycle Goal before the PR flips to `cycle-complete`.
- **Coverage is derived, never stored** — built-vs-unbuilt and the `US-### → Issue → PR → spec` map are computed at close-out from the Sprint Plan, Stories, Issues, and spec. There is no matrix file.
- **Deferred signal isn't lost, but doesn't pollute** — unresolved BLOCKED items are parked in the findings backlog (`.sprint/backlog.md`), never filed as GitHub Issues (Issues are committed work only).
- **Drift detection** — the spec-writer compares the Directory pointer-map and Stack against the filesystem and `package.json`. Catches changes made outside the plugin flow.
- **Commit before done** — both fixes and spec must be on disk before signalling completion.
- **Batch Bash** — combine independent `git`/`gh` reads into one invocation; keep mutating calls (`git commit`/`push`, `gh pr edit`, `gh issue create`) sequential. macOS/BSD-portable shell only.
