---
name: refine
description: "Fix review findings and update the system spec (.spec/spec.md). Use when participant has review comments on a PR, says 'fix these', 'address the review', 'refine my code', 'update the spec', 'sync the spec', 'close out this cycle', or wants to finalize work after /review."
argument-hint: "PR number or URL (optional — auto-detects current branch PR)"
---

# /refine — Fix Review Findings + Update Spec

You are closing out a development cycle. Phase 1: fix review findings from the PR. Phase 2: update the system spec to reflect what exists now.

You are mostly autonomous — one approval gate per phase (which findings to fix, then spec approval).

## Trust the artifact

The review findings you were handed are the **approved plan** — its decisions are already made and were already gated. Execute it as written. Do **not** re-derive it, re-validate it, re-confirm it, or re-present it for approval. A check whose normal outcome is "confirmed, proceed as written" is noise — don't run it.

Deviate **only** when faithful execution would *actually break*: a referenced file, symbol, or issue does not exist, or two instructions directly contradict. When you must deviate — **stop, amend the artifact with the reason** (edit it / comment it), then proceed. **Never diverge silently:** a change that isn't written back into the artifact makes the artifact lie, and a lying artifact is worse than none.

Findings were already scored and filtered — fix them, don't re-adjudicate validity. (The user chooses *which* to fix; that's their gate, not your re-litigation.)

**Initial request:** $ARGUMENTS

---

## Phase 1: Fix Review Findings

Read findings from GitHub, let user choose which to fix, dispatch implementer subagents, commit and push.

### 1. Find the PR

- If `$ARGUMENTS` contains a PR number or URL, use that.
- Otherwise, detect current branch PR: `gh pr view --json number,title,url`
- If no PR: "No PR found. Specify a PR number or URL."

### 2. Check for review completion

Before reading findings, verify that `/review` has actually run on this PR. The review phase posts a comment starting with `### Code Review` and swaps the PR label from `needs-review` to `needs-refine`.

```bash
gh pr view [number] --json comments --jq '.comments[].body'
```

Parse for structured /review findings using the format in `${CLAUDE_PLUGIN_ROOT}/skills/refine/formats/finding-format.md`. Look for the most recent comment matching the format (starts with `### Code Review`).

**If no `### Code Review` comment exists on the PR:** This PR hasn't been through `/review` yet. Tell the user: "This PR hasn't been reviewed yet. Run `/review` first — it runs the specialist agents and posts findings to the PR. Then come back to `/refine` to fix them and update the spec." Do not proceed.

### 3. Handle edge cases

- **Review comment says "No issues found":** "Review was clean. Nothing to fix — skipping to spec update."
- **Comments exist but no structured findings:** "I found comments but they don't match the /review format. Want me to read them and address manually, or skip to spec update?"

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

Then **dispatch all fix implementers in one parallel message** (sonnet) — the same shape `/build` uses for a wave — instead of read-then-dispatch one finding at a time. Each implementer gets the finding, its code context, and the suggested fix, via `${CLAUDE_PLUGIN_ROOT}/skills/refine/prompts/fix-prompt.md`.

Collect results: DONE → done; BLOCKED → report to user and continue with the rest. **File safety:** if two findings touch the same file, hand both to a single implementer so parallel fixes don't clobber each other.

### 6. Commit and push

```bash
git add [fixed files]
git commit -m "fix: address review findings from PR #[number]"
git push
```

---

## Phase 2: Update Spec

**Goal:** Update `.spec/spec.md` to reflect the system as it stands NOW.

### Determine mode

- **No `.spec/` directory** → Create it: `mkdir -p .spec`
- **No `.spec/spec.md` exists** → Creation mode (first cycle)
- **`.spec/spec.md` exists** → Update mode (subsequent cycle)

### Creation mode (first cycle)

The system has never been documented. Full exploration needed.

1. **Explore the codebase.** Dispatch 2-3 `codebase-explorer` agents (sonnet, parallel) to map:
   - Architecture (components, connections)
   - Data model (schemas, tables)
   - API surface (endpoints, contracts)
   - Patterns in use
   - Directory structure
   - Infrastructure/deploy target

2. **Gather additional context:**
   - Read `.prd/prd-v{latest}.md` (what was built this cycle)
   - Read `.adr/ADR.md` (decisions that shape the design)
   - Read `package.json` (dependencies = stack)

3. **Dispatch spec-writer agent** (sonnet) with:
   - Explorer results
   - PRD content
   - ADR content
   - `${CLAUDE_PLUGIN_ROOT}/skills/refine/formats/spec-format.md` (template)
   - Instruction: creation mode — fill all 7 sections

### Update mode (subsequent cycles)

The spec exists. Use the PR diff to scope what changed.

1. **Gather inputs:**
   - Read `.spec/spec.md` (current baseline)
   - Get PR diff: `gh pr diff [number]`
   - Read full content of files touched by the diff (not just hunks)
   - Get directory listing: `ls src/` (structural check)
   - Read `package.json` (dependency check)
   - Read `.adr/ADR.md` if it exists (any new decisions this cycle — skip if no ADR file)

2. **Dispatch spec-writer agent** (sonnet) with:
   - Existing spec
   - PR diff
   - Full touched files
   - Directory listing
   - package.json
   - ADRs
   - `${CLAUDE_PLUGIN_ROOT}/skills/refine/formats/spec-format.md` (template)
   - Instruction: update mode — only modify sections affected by the diff

### Review gate

After spec-writer returns:

1. Present the spec (or diff from previous version) to the user
2. Ask: "Here's the updated spec. Look good? (approve / edit)"
3. **Gate:** User must approve before commit

If user wants edits: apply their changes, then proceed.

### Commit spec

```bash
git add .spec/spec.md
git commit -m "docs: update spec after PRD v{N} cycle"
git push
```

---

## Phase 3: Close Cycle

### 1. Update cycle label

Mark the PR as cycle-complete — all phases have run:

```bash
gh pr edit [number] --remove-label "needs-refine" --add-label "cycle-complete"
```

### 2. Present summary

```
## Cycle Complete: PR #[number]

**Fixes:** [N] applied, [N] skipped, [N] blocked
**Spec:** [created | updated] — .spec/spec.md

The full cycle ran: /plan → /tickets → /build → /review → /refine.
Next cycle: run `/plan` to start planning the next feature.
```

---

## Key Principles

- **You are the orchestrator** — subagents fix code and write the spec. You coordinate.
- **Session-boundary safe** — reads everything from disk/GitHub. Works in a fresh session.
- **Spec follows template** — 7 sections, same structure every time. The spec-writer agent enforces this.
- **Diff-driven updates** — don't re-describe the whole system. Only update what changed.
- **Drift detection** — the spec-writer compares its sections against the filesystem. Catches changes made outside the plugin flow.
- **Commit before done** — both fixes and spec must be on disk before signalling completion.
- **Batch Bash** — combine independent `git`/`gh` reads into one invocation; keep mutating calls (`git commit`/`push`, `gh pr edit`) sequential. macOS/BSD-portable shell only.
- **User controls both gates** — which findings to fix, and whether the spec looks right.
