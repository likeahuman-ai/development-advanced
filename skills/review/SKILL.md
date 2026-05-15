---
name: review
description: "Review a PR with specialist agents and confidence scoring — surfaces only high-confidence findings. Use when participant has a PR ready, says 'review my code', 'check this PR', 'is this ready', 'code review', or has an open pull request that needs specialist review."
argument-hint: "PR number or URL (optional — auto-detects current branch PR)"
---

# /review — PR Review

Follow the communication tone in `${CLAUDE_PLUGIN_ROOT}/skills/review/references/tone.md`.

You are reviewing a PR with specialist agents and confidence-based scoring. You combine deep specialist analysis with aggressive noise filtering — only findings above confidence threshold reach the user (65% user-facing, 80% internal).

You are mostly autonomous. No gates — run the full pipeline and present results.

**Initial request:** $ARGUMENTS

---

## Phase 1: Eligibility

**Goal:** Find the PR and check if it's worth reviewing. Use Haiku-level reasoning — this is a yes/no decision.

### 1. Find the PR

- If `$ARGUMENTS` contains a PR number or URL, use that.
- Otherwise, detect the current branch's PR: `gh pr view --json number,title,state,isDraft,additions,deletions,files`
- If no PR found, tell the user: "No PR found for the current branch. Specify a PR number or URL."

### 2. Check eligibility

Skip the review (tell the user why) if:
- PR is closed or merged
- PR is a draft (suggest: "This PR is still a draft. Run `/review` again when it's ready.")
- PR has 0 changed files
- PR changes only lock files, generated files, or non-code assets

Otherwise, proceed.

### 3. Gather PR context

```bash
gh pr view [number] --json number,title,body,headRefName,baseRefName,additions,deletions
gh pr diff [number]
```

Get the full SHA for code links: `git rev-parse HEAD`

---

## Phase 2: Summarize

**Goal:** Understand what changed and determine which specialists to run. Haiku-level reasoning.

### 1. Categorize changed files

Read the diff and classify each file:
- **TypeScript source** — triggers code-quality-reviewer
- **Error handling code** (try/catch, .catch, error callbacks) — triggers silent-failure-hunter
- **Type definitions** (.types.ts, interfaces, type aliases) — triggers type-design-reviewer
- **Test files** (.test.ts, .spec.ts) — triggers test-coverage-reviewer
- **Files with code comments** (JSDoc, inline comments) — triggers comment-analyzer
- **Files with high git churn** (check `git log --oneline -10 -- [file]`) — triggers history-reviewer
- **Security-sensitive files** — triggers security-reviewer:
  - `.env`, `.env.*` files in the diff
  - Config/settings files (`config.ts`, `*.config.*`, `settings.*`)
  - Files containing string literals matching key patterns: `AKIA`, `sk_`, `sk-`, `ghp_`, `Bearer`, `-----BEGIN`, `password`, `secret`, `token` as assigned values (not env var references)
  - Test fixtures with hardcoded data objects containing email-like, phone-like, or name-like values
  - Files with `console.log`/`console.error`/`logger.*`/`throw new Error` containing interpolated user data
  - API route handlers processing request bodies

### 2. Detect platform and inject context

Identify the project platform (e.g., Next.js, VS Code extension, CLI tool) from package.json, file structure, and framework markers. If a known platform is detected, inject the appropriate context into the `{{platform_context}}` slot in the review dispatch prompt (`skills/review/references/review-prompt.md`).

### 3. Check for coding standards

Before building the roster, check if the participant has coding standards installed:

1. Check if `~/.claude/skills/coding-standards/SKILL.md` exists.
2. If it exists, read the "Read when..." table in that file. Map changed file categories to rule files:
   - TypeScript source → `rules/typescript-quality.md`, `rules/types-and-constants.md`
   - React/JSX components → `rules/react-patterns.md`, `rules/component-architecture.md`
   - Tailwind/styling → `rules/tailwind-and-tokens.md`
   - Convex files → `rules/convex-backend.md`
   - Node.js backend → `rules/nodejs-backend.md`
   - State management → `rules/state-management.md`
   - Any code → `rules/general-quality.md`, `rules/naming-conventions.md`
3. Read only the matched rule files (2-4 files, not all 14). Store the content for injection into the `standards-reviewer` dispatch prompt.
4. If no coding standards file exists, skip — do not dispatch `standards-reviewer`.

### 4. Build the specialist roster

Always include:
- `code-quality-reviewer` (sonnet)
- `code-simplifier` (inherit) — runs after others

Conditionally include based on file classification above:
- `silent-failure-hunter` (sonnet)
- `type-design-reviewer` (inherit)
- `test-coverage-reviewer` (sonnet)
- `comment-analyzer` (sonnet)
- `history-reviewer` (sonnet)
- `security-reviewer` (sonnet) — if security-sensitive file patterns detected
- `standards-reviewer` (sonnet) — if coding standards exist (Step 3 above found rule files)

---

## Phase 3: Specialist Review

**Goal:** Run specialist agents in parallel and collect findings.

**HARD RULE — You are the orchestrator, NOT the reviewer.**

You MUST NOT write review findings yourself. All findings come from dispatched specialist agents. If you catch yourself about to analyse the diff and write findings — STOP. That work belongs to the subagents.

**Allowed tools during Phase 3:**

| Tool | Allowed | Purpose |
|------|---------|---------|
| Agent | YES | Dispatch all specialist review agents |
| Read | YES | Loading review prompt template, reading agent results |
| Grep / Glob | YES | File classification for roster decisions |
| Edit / Write | NO | No file modifications during review |

### 1. Dispatch agents

Load `skills/review/references/review-prompt.md` for the dispatch template. You MUST call the Agent tool for each specialist in the roster. Launch all independent specialists in a **single message with multiple Agent tool calls** for parallel execution.

**Dispatch enrichment:** When dispatching the `security-reviewer`, read `skills/review/references/security-detection-guide.md` and include its content in the Agent prompt alongside the standard review-prompt.md template. This gives the agent the detection heuristics and PII taxonomy it needs.

**Standards enrichment:** When dispatching the `standards-reviewer`, inject the pre-selected coding standards rule content (gathered in Phase 2, Step 3) into the Agent prompt. Do NOT tell the agent to read files — provide the rule content directly. The agent receives concrete rules, not file paths.

For each agent, provide in the Agent prompt:
- PR context (number, title, description)
- The relevant portion of the diff (scoped to the agent's focus area)
- Changed file list
- Clear instruction to review only changed code

```
Agent tool calls (all in one message for parallel execution):

  Agent 1:
    description: "Review #[number] code quality"
    model: "sonnet"
    prompt: [review prompt with code-quality-reviewer focus + relevant diff]

  Agent 2:
    description: "Review #[number] silent failures"
    model: "sonnet"
    prompt: [review prompt with silent-failure-hunter focus + relevant diff]

  Agent 3:
    description: "Review #[number] type design"
    prompt: [review prompt with type-design-reviewer focus + relevant diff]

  ... (one per specialist in the roster)
```

Do NOT review the code yourself. Do NOT "quickly check" one area because it seems simple. Every specialist gets a subagent.

### 2. Dispatch code-simplifier last

After all other agents return, you MUST call the Agent tool for the code-simplifier. Provide in the Agent prompt:
- The full diff
- Findings from other agents (so it doesn't duplicate their work)

### 3. Collect all findings

Gather findings from all agent results. Each finding should have:
- Description
- File path and line number
- Evidence (code snippet)
- Which agent found it
- Suggested fix

---

## Phase 4: Confidence Scoring

**Goal:** Score each finding and filter out noise.

**HARD RULE — You MUST dispatch scoring to a subagent.**

You MUST NOT score findings yourself. Dispatch a single scoring agent via the Agent tool that evaluates all findings in one pass.

### 1. Score each finding

You MUST call the Agent tool with `model: "haiku"` to score all findings. Provide in the Agent prompt:
- All findings from Phase 3 (description, file, line, evidence, agent, suggestion)
- The scoring rubric below
- The PR diff for verification

```
Agent tool call:
  description: "Score #[number] review findings"
  model: "haiku"
  prompt: [all findings + scoring rubric + diff]
```

**Scoring rubric (0-100):**
- Is the evidence specific — file, line, code snippet? (+20)
- Is the issue in code the PR actually changed? (+20)
- Would a senior engineer flag this? (+20)
- Is this a real bug or a preference? (+20 for real bug)
- Could CI catch this instead? (-20 if yes)

Scores:
- **0** — false positive, doesn't hold up
- **25** — might be real, might be false positive
- **50** — real but minor, nitpick territory
- **75** — verified real, will impact functionality
- **100** — certain, confirmed with evidence

### 2. Classify each finding

Before applying the threshold, classify each finding:
- **User-facing** — visible UI bugs, broken buttons, data loss, broken user flows, visual regressions
- **Internal** — code quality, type safety, style, performance, error handling patterns

**Security finding classification:** SECRET and PII findings from the security-reviewer are **user-facing** (threshold: 65) — these represent real data exposure risk. LOG_LEAK and INTERNAL_URL findings are **internal** (threshold: 80) — these are code hygiene issues.

### 3. Filter (two-tier threshold)

Apply a two-tier threshold:
- **User-facing findings:** Drop below **65**. These are real bugs participants will see.
- **Internal findings:** Drop below **80**. This is the noise filter.

This recovers real user-facing bugs that scored 55-79 while keeping internal noise filtered.

### 4. Categorize survivors

- **Critical** (90-100) — must fix before merge
- **Important** (65-89 user-facing, 80-89 internal) — should fix
- **Suggestions** — improvement opportunities above threshold

---

## Phase 4.5: Capture Deferred Findings

**Goal:** Preserve middle-band signal for the next planning cycle. This phase is SILENT — the participant sees nothing.

After scoring and filtering, collect all findings that scored 50-79 (dropped by the threshold but verified as real by the scoring agent).

**If no findings in the 50-79 range:** Skip this phase entirely. Proceed to Phase 5.

**If deferred findings exist:**

### 1. Check for existing deferred PRD

```bash
# Look for a file with status: deferred in .prd/
grep -l "status: deferred" .prd/prd-*.md 2>/dev/null
```

### 2. Determine version number

Read `.prd/` directory, find the highest existing version number N across ALL files (regardless of status — draft, built, released, archived, deferred all count). The deferred file will be `prd-v{N+1}.md`. This prevents collisions with existing drafts or other files.

### 3. Write or append

**If a deferred PRD already exists:**
- Read the existing file
- Append a new section for this PR's findings
- Deduplicate by file+line against existing entries (keep higher score)

**If no deferred PRD exists:**
- Create `.prd/prd-v{N+1}.md` with the deferred format below

### 4. Deferred PRD format

```markdown
---
version: {N+1}
status: deferred
date: {today}
author: /review
previous: prd-v{N}.md
---

# Deferred Findings

Review findings that scored 50-79 — real but below the noise threshold. These inform the next planning cycle.

## PR #{number} — {title} ({date})

### Pattern: {agent-name} ({count} findings)

| Score | File | Finding | Suggestion |
|-------|------|---------|------------|
| 72 | src/api/handler.ts:45 | Error caught too broadly | Narrow catch to specific error types |
| 65 | src/utils/logger.ts:12 | User object logged without redaction | Log userId instead of full user object |
```

Group findings by agent to surface patterns. If one agent flags 4 similar issues, that's a pattern worth planning for.

### 5. Commit silently

```bash
git add .prd/prd-v{N+1}.md
git commit -m "docs: capture deferred review findings for next cycle"
```

**One-deferred rule:** Maximum ONE `status: deferred` file per `.prd/` directory. Always append to existing rather than creating a second.

**No output to participant.** This entire phase produces no visible output. The PR comment and presentation in Phase 5 proceed as if this phase didn't run.

---

## Phase 5: Report

**Goal:** Comment on the PR and present findings to the user.

### 1. Format the PR comment

If findings survived scoring:

```markdown
### Code Review

Found [N] issues:

1. **[Critical/Important]** [brief description] — found by [agent name]

   https://github.com/[owner]/[repo]/blob/[FULL-SHA]/[path]#L[start]-L[end]

2. **[Critical/Important]** [brief description] — found by [agent name]

   https://github.com/[owner]/[repo]/blob/[FULL-SHA]/[path]#L[start]-L[end]

---

Reviewed by: [list of agents that ran]
Confidence threshold: 65/100 user-facing, 80/100 internal
```

If no findings survived:

```markdown
### Code Review

No issues found. Reviewed for: [list what was checked based on agents that ran].

Confidence threshold: 65/100 user-facing, 80/100 internal
```

### 2. Post the comment

```bash
gh pr comment [number] --body "[comment]"
```

### 3. Present to user

Show the user:
- How many findings each agent produced vs how many survived scoring
- The surviving findings grouped by severity
- Which agents ran and what they checked

```
## Review: PR #[number] — [title]

**Agents:** [list] | **Findings:** [N raw] → [M after scoring]

### Critical
- [finding with file:line]

### Important
- [finding with file:line]

### Clean Areas
- [what was checked and found clean]
```

---

## Phase 6: Handoff to /refine

After presenting findings, direct the participant to GitHub and suggest the next step:

> "I've posted the findings to your pull request — go have a look at the comments: [PR URL].
>
> When you're ready to address them, run `/refine` — it'll let you choose which findings to fix and handle them with dedicated agents."

**Do not offer to fix findings yourself.** The /refine skill handles this with structured subagent dispatch. Do not inline any fixes in this skill.

---

## Key Principles

- **You are the orchestrator** — you coordinate, you do not review or score. Every specialist and the scoring phase get a subagent via the Agent tool. No exceptions.
- **Parallel dispatch** — launch all independent specialists in a single message with multiple Agent tool calls. This is the entire point of the multi-agent architecture.
- **Only real issues** — the two-tier threshold exists to prevent noise while catching user-facing bugs. Trust it.
- **Evidence required** — no finding without file:line and code snippet.
- **Changed code only** — never flag pre-existing issues.
- **No CI duplication** — don't flag what linters, typecheckers, or tests catch.
- **Cheap where possible** — Haiku for scoring, sonnet/inherit for actual review.
- **Simplifier runs last** — it benefits from seeing other agents' findings to avoid overlap.
- **Full SHA in links** — abbreviated SHAs break GitHub links.
