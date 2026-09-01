---
name: sprint-refine
description: "Closes out the sprint: every reviewed PR ends landed on the team's shared development trunk with its published findings fixed, and the spec/ADR knowledge layer ends current at sprint close — the close of team-scale parallel building. Use when, on that trunk/team repo, PR(s) carry needs-refine, a reviewed PR's published findings await fixing, or the user says 'fix these', 'address the review', 'refine my code', 'refine the sprint', 'land the sprint', 'merge the PRs', 'ship it', 'close out the sprint', 'update the spec', or 'sync the spec'. A solo, sequential, single-PR project belongs to the sibling development plugin instead."
argument-hint: "PR number(s) or URL(s) (optional — defaults to `needs-refine` label discovery)"
---

# /sprint-refine — Fix, Patch the Spec, Land

Phase 5 of the sprint flow (Plan → Tickets → Build → Review → Refine). The spine: **per PR, fixes → land (code only)**; then **one `.spec` sweep at sprint close** — a single `spec-writer` pass over the whole sprint's landed diff, accuracy-verified, carrying the plan flip. Each PR gets its findings fixed and lands on `development` by rebase + fast-forward (the flow never merges). Runs on plain git + GitHub via `gh`. Starts at PR discovery (5.0.1), ends at the summary (5.3.5). Promoting `development → main` is a separate release step — out of scope.

The latest `### Code Review` comment is this phase's **work order** — tickets drive Build, comments drive Refine. Never re-run Review's work, never re-check Tickets' or Build's decisions (*trust the artifact*).

This skill's own prompt and format live at `${CLAUDE_PLUGIN_ROOT}/skills/sprint-refine/prompts/fix-prompt.md` and `${CLAUDE_PLUGIN_ROOT}/skills/sprint-refine/formats/spec-format.md`. Cross-skill contracts resolve to their owners: `finding-format` (Review) at `${CLAUDE_PLUGIN_ROOT}/skills/sprint-review/formats/finding-format.md`; `commit-format` and `implementer-prompt` (Build) at `${CLAUDE_PLUGIN_ROOT}/skills/sprint-build/formats/commit-format.md` and `${CLAUDE_PLUGIN_ROOT}/skills/sprint-build/prompts/implementer-prompt.md`.

**Initial request:** $ARGUMENTS

## You are the orchestrator, NOT the fixer

Discover the PRs, present findings, dispatch fix agents and `spec-writer`, author the commits, run the verify gate, land, clean up — never write a line of code or spec prose yourself (the lone exception: in creation mode, transcribing `spec-writer`'s returned founding `.spec/spec.md` to disk, since it has Edit not Write — that is transcription, not authoring). If you catch yourself editing source to fix a finding — STOP; that work belongs to a dispatched agent (*subagents are hands, not authors* — they return status + summary + working directory, never a diff; the session collects their worktrees and is the sole committer). Every dispatch runs Sonnet or Opus, never Haiku.

| Tool | Allowed | Purpose |
|------|---------|---------|
| Agent | YES | Fix agents (5.1.3, 5.1.5) and the one `spec-writer` sweep (5.3.1) |
| Bash (`git`, `gh`, the build-order provision/Verify commands) | YES | Discovery, positioning, collection commits, the verify gate, publishes, the land, cleanup |
| Read / Grep / Glob | YES | Work orders, build-order, review-standard slices, plan status |
| Edit / Write | ONLY the 5.3.1 plan flip + creation-mode spec write | The `.sprint` `draft → built` status edit (rides the spec sweep), and — in creation mode only — writing `spec-writer`'s returned founding `.spec/spec.md` to disk — nothing else |

Subagents never run git — all vcs motion (collection commits, cherry-picks, rebases, pushes) belongs to the session. The parallel writers each work inside their own isolated git worktree, isolation declared in their agent-file frontmatter: the `fix-agent` custom subagent (5.1.3) writes each target file's fixes, and the one `spec-writer` sweep (5.3.1) writes the `.spec` delta (update mode; in creation mode it returns the full file for the session to write). The solo-heal / marker-fix at a halted operation (5.1.4) is the exception — a **general-purpose** dispatch that resolves the markers **in place on the main tree** (no worktree, no isolation). All write files; the session owns all vcs motion.

**One human gate:** 5.2.1 (accept the PR against its DoD) — the genuine ship/no-ship call the model can't make for the user. The fix-set (5.1.2) and the `.spec` sweep (5.3.1) run **autonomously**: 5.1.2 defaults to fix-all-published (the published set *is* Review's arbitrated must-fix set), and 5.3.1 self-verifies the sweep's accuracy against the landed code — gating either would rubber-stamp work the model can verify itself, which the rubric's own gate-ergonomics criterion forbids. The vcs motion after the gate runs autonomously — never gate a commit, a push, or the land itself.

**Parking is not solving.** A conflicted land rebase is aborted — `git rebase --abort` restores the branch byte-clean — and the PR parks (`conflict-parked`) for a **paired human session**; this skill never auto-resolves a land conflict, and the trunk never stalls: every other PR proceeds.

---

## 5.0 Phase setup

Goal: find the sprint's PR(s) under refine, then fan out per PR.

### 5.0.1 Find the sprint's PR(s)

- If `$ARGUMENTS` names PR number(s) → use exactly those, fetching each PR's metadata (`gh pr view <n> --json number,title,url,headRefName,headRefOid,baseRefName,labels`) — the args path must produce the same fields the discovery query does.
- Else discover by label — the durable discovery key (a fresh session holds no branch state to infer from):

```bash
gh pr list --label needs-refine --state open --json number,title,url,headRefName,headRefOid,baseRefName,labels
```

if nothing found → stop and tell the user: no open PR carries `needs-refine` — pass a PR number, or run `/sprint-review` first. (`baseRefName` is the stack-detection input — 5.0.2's land ordering and 5.2.2's stacked path branch on each PR's `base:`.)

Also read the sprint's build-order issue (an inline inspection feeding 5.1.3 and 5.1.5 — labels `build-order` + `v{N}`, closed at Build 3.4.3; derive `v{N}` from the `feat/sprint-v{N}(-<g>)` head refs): its `## Verify` section carries the **provision** and **Verify** commands.

```bash
gh issue list --label build-order --label "v{N}" --state closed --json number,title
gh issue view <number>
```

### 5.0.2 Iterate per PR — fixes fan out, the land serialises

**Fixes fan out across PRs.** Each PR's fix-agents run in their **own git worktrees** (native isolation — Build 3.1.2's discipline), so the **slow work overlaps across PRs**. Mechanism: native isolation cuts each worktree from the session's `HEAD`, so position the checkout at a PR's head (5.0.3) and dispatch *that* PR's fix batch — then move to the next PR and dispatch its batch; **once cut, the worktrees are independent, so all PRs' batches run concurrently** (creation is staggered, execution overlaps). The session stays **sole committer**: as each batch returns, it re-checkouts that PR's branch and collects each worker serially onto it (collection is cheap; the dispatched agent work is what parallelises).

**The spec is NOT per PR.** Per PR the flow only **fixes and lands code** — it never touches `.spec`. The whole sprint's spec is a **single `spec-writer` sweep at sprint close (5.3.1)**, run once over the sprint's landed diff after the PRs have landed. This is the deliberate trade for one dispatch: the trunk spec lags the code by at most one refine run — a lag `spec-format`'s `last_updated`/`sprint` frontmatter is built to signal — in exchange for the spec-writer seeing the system **as it now IS**: no per-PR fan-out, no forward/PR-relative narration, no cross-PR spec reconcile.

**Land order.** **Peers** (`base: development`) are independent → land in **any order**. A **stack** (a PR whose `base:` is a prior in-sprint group) lands in **dependency order, base first** (5.2.2) — the dependent rebases onto the *landed* base. The land is **always serial** regardless: `development` is one resource and the non-fast-forward push rejection is the concurrency control (5.2.2).

### 5.0.3 Re-assert position (per PR)

Sync in first — review's push may be from another session:

```bash
git fetch origin
```

Then set the work surface — check out this PR's head branch and confirm it sits at the PR's published head:

```bash
git checkout <headRefName>          # or: git checkout -b <headRefName> origin/<headRefName> when no local branch exists
git rev-parse HEAD                  # matches the PR's headRefOid; behind origin → git merge --ff-only origin/<headRefName>
```

a mismatch that `--ff-only` cannot resolve → surface it, never force. The fixes collect onto this branch; native isolation cuts each fix worktree from `HEAD` (Build 3.1.2's basing discipline) — the checkout must sit here, clean, at dispatch.

if `git status --porcelain` shows foreign uncommitted state → surface it to the user, don't blind-switch over another session's work.

### 5.0.4 Check eligibility (per PR)

Read the **latest** `### Code Review` comment on this PR — it is this PR's work order (`gh pr view <number> --json comments`, take the newest comment carrying the marker):

- if no `### Code Review` comment → not reviewed → skip this PR (record why for the end-of-run report, 5.2.3).
- if already refined → skip **5.1 only**, straight to 5.2 — the land still runs. **Already refined =** the PR's chain carries `fix` commits postdating the latest `### Code Review` comment whose trailers resolve to that comment's findings (`git log origin/development..origin/<headRefName>` — the per-worker fix commits are the evidence; only 5.2.3 removes `needs-refine`, so a fixed-but-unlanded PR from a dead run must still land or it re-skips forever). A full PR skip is only for closed PRs or PRs already on the trunk.
- if zero published findings → skip 5.1 entirely, straight to 5.2 — the land still runs.
- if the PR carries `conflict-parked` → straight to 5.2.2 to re-attempt the land: the abort left no local conflict state (both sides live published), so the re-attempt is a fresh fetch → rebase — the trunk may have moved and the conflict dissolved; if it halts again → it still awaits the paired session — abort again, report, and continue with the other PRs. A heal-resume that lands the sprint's **last** PR makes this run's 5.3.1 sweep the one that carries the `draft → built` flip.
- if the PR is a stacked dependent (`base:` ≠ `development`) whose base group has **not landed** → its fixes (5.1) may run, but its **land defers**: if the base is still queued this run → process the base first (5.2.2's base-first order); if the base **parked** → park the dependent too (`conflict-parked` + a details comment naming the blocked-on base PR — no rebase is attempted, so there is nothing to abort), same paired-session exit; it re-enters through this step's parked route once the base lands.

---

## 5.1 Fixes (per PR)

Goal: apply this PR's must-fix findings — the published set in its comment (the findings backlog stays in the repo, untouched here).

### 5.1.1 Present findings

Present this PR's published findings from the latest `### Code Review` comment (shaped per `finding-format`) — scores + `testable` tags exactly as published, no re-scoring, no severity labels.

### 5.1.2 Select the fix-set (autonomous)

**Default: fix all published findings.** The published set already *is* Review's Opus-arbitrated must-fix set (`≥75` ∨ `≥50`∧testable), so "which to fix" is a computable default, not a decision needing a human — gating it would rubber-stamp work the model can verify itself. Narrowing the fix-set is **opt-in**, never a blocking stop: if the user has *already* asked to defer a tier, honour it; otherwise proceed with fix-all-published. if the published set is empty → straight to 5.2 (the land still runs; distinct from 5.0.4's zero-findings skip).

### 5.1.3 Dispatch fixes (parallel)

**First, group the picked findings by target file.** Read each picked finding's target file from its `finding-format` field — the published comment's `Edits:` edited-path list, the authoritative source for this partition. Then the **unit of dispatch is the file, not the finding**:

- **One worker per distinct target file** — not one per finding. Partition the picked set by `Edits:` and fan out exactly one fix worker for each distinct file.
- **A file carrying multiple findings is ONE worker** — that worker runs all of that file's findings within itself (serial within-file). Inject the per-finding payload for **each** finding the worker owns: description · evidence · permalink · its `Expected:` oracle when present · its review-standard slice — **sourced from the finding's `Violates:` line** when present (resolve the pointer: the named `ADR-###` Y-statement, the `.spec` anchor's section, or the named rule's content), else derived from the finding's description (the stated fallback) · the `testable` → regression-test rule (the test encodes the `Expected:` line). Also inject the coding-standards pointer (the user's conventions, if installed — as Build 3.2.1 does).
- **A finding spanning multiple files** reduces to within-worker serial on the involved files — note it so the same finding isn't double-dispatched across those files' workers.

Partitioning by file makes parallel workers touch disjoint files by construction — the §5.1.4 collection-conflict heal drops to the safety net (it was the routine path before the partition).

Dispatch fix agents in ONE parallel batch — one **`fix-agent` custom subagent** per distinct target file, briefed with `fix-prompt`, its worktree cut from this PR's head (5.0.3 positioned the checkout there — native isolation bases on `HEAD`). **Record `git rev-parse HEAD` as this batch's dispatch base** — on a 5.1.5 repair round the branch tip has moved past the published head, so the recorded value, not the PR's `headRefOid`, is what the 5.1.4 isolation check asserts against. **Isolation is not passed here: the `fix-agent` agent file carries `isolation: worktree` (and `effort`) in its frontmatter, so every spawn is worktree-isolated deterministically — never a per-call flag the session can drop.** Each dispatch resolves a **pinned capable tier from the published `Cost:` markers** — the worker's tier is the highest cost among the findings it owns: `S`/`M` → sonnet · `L` → opus (never Haiku, no silent inherit, no session discretion) — passed as the per-dispatch `model` override; the fix commit's `Assisted-by:` trailer (5.1.4) is the tier record.

**The Build 3.2.1 worktree contract applies in full — state it in the prompt:**

- the isolated copy is a linked git worktree of the team repository — **never run git** (git there mutates shared refs and history); just write files
- it ships **no project deps** → run the build-order's **provision** command first, then its **Verify** command, and self-verify to green — never rely on JS's leaking parent `node_modules` (fragile, unsound on dep changes)
- **return status + summary + working directory** — no diff; the session commits the worker's tree and cherry-picks it

### 5.1.4 Collect the fixes (per worker)

Barrier — collect every fix-agent's **status + summary + working directory**. **Verify isolation engaged first — fail-loud (Build 3.2.3's check):** each `SUCCESS` worker's reported directory is a linked worktree of this repo — `git -C "<worker-dir>" rev-parse HEAD` equals the batch's recorded dispatch base (5.1.3) and `--git-common-dir` resolves into this repo's `.git` — never the repo root; a silent no-op isolation → surface, discard the commingled edits, re-dispatch. Then the session collects **one atomic commit per fix worker — its distinct target file** — onto this PR's branch, `fix(scope): <the file's findings>`:

```bash
git -C "<worker-dir>" add -A
git -C "<worker-dir>" commit -m "<message + trailers per commit-format>"
git cherry-pick <that commit's SHA>       # main checkout, HEAD on this PR's branch — appends the fix commit
```

Message worker-suggested, session-owned; trailers per `commit-format` — pointers to artifact-owned facts by logical ID, never copies — **each read off its own published line when present**: a finding's `Ticket:` line forwards as the `Ticket:` trailer, its `Violates: ADR-###` as the `ADR:` trailer; a line absent or non-forwardable (a `Violates:` naming a `.spec` anchor or rule carries no ADR) → derive that trailer at collection time from the PR chain's commits on the finding's edited paths (the stated fallback, no longer the routine path). The worker's commit carries **every finding it owned**: each fix plus its regression test when `testable` (the test proves the fix; a single-finding file reads one-commit-per-finding — the common case). Never bundle findings from **different** files into one commit.

**Safety net — residual overlaps only.** §5.1.3 partitions the batch by target file, so parallel workers touch disjoint files by construction (a declared spanning finding stays with one worker — the 5.1.3 spanning rule) and a same-file collision should not occur on the normal path. This heal stays only for the residual overlap the partition couldn't foresee — e.g. a finding whose true edit set wasn't fully captured by its target-file field. When such an overlap does surface → Build 3.2.4's mechanism: the cherry-pick **halts at a controlled halt** (markers in the tree, nothing committed) → dispatch a **general-purpose solo-heal subagent** (`implementer-prompt` Mode B — **not** the isolated `fix-agent` custom subagent) to resolve the markers on the main tree, then `git add -A && git cherry-pick --continue` — message + trailers ride unchanged (*hands-not-authors*: the session writes no code). Two heal rounds per conflict, then escalate.

Tail — remove the fix worktrees (spent resource, Build 3.2.6's mechanics): `git worktree remove --force "<worker-dir>"` + `git worktree prune` (+ `git branch -D` any temp worker branch the harness cut). The harness auto-cleans only unchanged worktrees, so the session runs this explicitly; idempotent. The 5.3.4 sweep is only the safety net.

### 5.1.5 Verify the fixed tip

**The explicit gate.** Push is NEVER the gate; the gate is this step. The session re-runs the build-order **Verify** command on this PR's fixed tip AND asserts no conflict state survived collection:

```bash
<the build-order Verify command>
git status --porcelain      # empty — no unmerged paths, no operation in progress
```

The fixes are fresh code — the re-run certifies their form on the integrated tip. The gate checks form only; it never executes the regression tests, which land with the fixes for the corridor to run post-land.

if Verify fails → the repair loop: dispatch the **`fix-agent` custom subagent** off the fixed tip (its frontmatter gives the fresh worktree; the `fix-prompt` shape applies with its slots filled for a **repair** — the failing Verify output is the sole work order in the findings slot, `Testable: no`, the target file(s) read from the failure itself; no arbitrated-finding framing, no `Edits:` partition) → the session collects it (5.1.4 mechanics) → re-gate. The retry's tier is deterministic too: a failure attributable to one worker's fix inherits **that worker's resolved tier**; a cross-finding or unattributable failure dispatches on **Opus** — never Haiku, never a silent inherit. **Bounded (two rounds), then escalate** — never loop indefinitely; a repeating failure is a signal for the user, not something to paper over.

### 5.1.6 Push

Publish the verified tip — push only verified state (5.1.5 just certified it):

```bash
git push origin <headRefName>
```

---

## 5.2 Land (per PR)

Goal: accept each PR against its DoD and land it on `development` — **code only** (the spec's one home is 5.3.1).

### 5.2.1 Gate — accept this PR

The user gives go/no-go on this PR against the **per-PR DoD** — the `.brief` DoD applied to what this PR delivers. Acceptance gates the content; the land op follows autonomously — never gate the land itself. On a **no-go** → the PR stays open under `needs-refine`, the user's reason is recorded as a PR comment, and the run continues with the other PRs (the end-of-run report names it).

### 5.2.2 Land this PR

```bash
git fetch origin development
git checkout <headRefName>
git rebase origin/development
```

**The standing gate rides every land.** if this PR reached 5.2 without a 5.1.5 run on its current tip (the zero-published-findings and already-refined skip paths) → run the build-order **Verify** on the rebased tip now, before anything publishes — green proceeds; red routes to 5.1.5's fail branch. No PR reaches the trunk without one session-run gate on the state that lands.

**Clean rebase** → publish the rebased head, then fast-forward the trunk:

```bash
git push --force-with-lease origin <headRefName>   # the rebased PR head — an ephemeral feature branch, never the trunk
git push origin <headRefName>:development           # the FF land — the remote development fast-forwards to the PR tip
```

Updating the PR head first means the landed commits are the PR's own commits, so GitHub records the PR merged when they reach `development`. The trunk push is forward-only by construction — the branch was just rebased onto the fetched trunk; if the push still rejects as **non-fast-forward** (a concurrent land moved the trunk) → that rejection IS the concurrency control: fetch · re-rebase · retry. The PR then closes — its commits landed under fresh SHAs; identity across the rebase rides subjects + trailers. if this PR carried `conflict-parked` (a re-attempted land whose conflict dissolved) → remove it now: `gh pr edit <number> --remove-label conflict-parked` — the label means awaiting-resolution, and the completed land is the resolution.

**Stacked PRs — base-first, retarget before delete.** When a PR is a stack (its `base:` ≠ `development`):

- Land the **base group first** — it lands as a normal peer onto `development` (above). **Before the base's rebase moves anything, record its pre-land tip:** `git rev-parse <base-headRefName>` — carry that SHA for the base's dependents; 5.2.3 deletes the base's branches, so nothing else survives to derive it from (`git merge-base` resolves to the sprint fork point once the base lands rebased). Only then land each dependent, in `base:` order (its PR base was already retargeted to `development` by the base's 5.2.3 cleanup — the guard's single home).
- Land the dependent: `git fetch origin development` · `git rebase --onto origin/development <recorded base pre-land tip> <headRefName>` (only the dependent's own commits move — the base group's commits are already on `development`; subjects + trailers preserved, **never a squash**) · **re-run the build-order Verify on the rebased tip** (its base changed — a fix that landed on the base may have shifted a seam) · force-with-lease the head + FF push as above.
- A conflict at the dependent's restack is the same path below — a **code** conflict parks (per PR lands only code, so that is the only kind here).

**Conflicted rebase → abort and park, don't block — and NEVER auto-resolve:**

```bash
git rebase --abort                                   # restores the branch byte-clean — no conflicted or half-applied state survives
gh pr edit <number> --add-label conflict-parked
gh pr comment <number> --body "<the conflicting files · the commit the rebase halted on · the landed sprint it crosses · @both devs>"
```

- suspend this PR's land and **skip 5.2.3 for this PR entirely** — cleanup is for landed PRs only, and no branch deletion ever touches a parked PR; continue with the next PR at 5.0.3. Everything else proceeds. Parked state is recomputable — both sides stay published (the PR head on origin, the trunk on origin) and the abort deleted nothing.
- resolution is a **paired human session** — not this skill: the pair re-runs the land (fetch → rebase), resolves the markers together at the halt, `git rebase --continue` (message + trailers ride unchanged), **re-verifies the healed tip**, then resumes the land (force-with-lease the head → FF push the trunk) → remove `conflict-parked`.

### 5.2.3 Clean up (per PR)

**Landed PRs only.** A parked PR skipped this step at 5.2.2's park exit, and a no-go PR (5.2.1) skips it too — their branches are never deleted; for either, continue with the next PR (5.0.3).

**Before deleting this PR's branch:** retarget every open PR whose `base:` is this branch to `development` (`gh pr edit <dependent> --base development`) — a dependent stacked PR must never point at a branch about to vanish (GitHub orphans it); the dependent's own land at 5.2.2 rebases and re-verifies it regardless.

Then delete the landed PR's branches, local + remote — the rebase-land never uses the merge button, so GitHub's auto-delete never fires; the session deletes them itself:

```bash
git branch -D <headRefName>
git push origin --delete <headRefName>
```

Then remove the PR's flow labels (`gh pr edit <number> --remove-label needs-refine`).

if PRs remain → next PR (back to 5.0.3). Once every PR is processed → **5.3**: the one spec sweep runs over what landed this run, then the sprint wraps. if any PR is parked or skipped → the sweep still runs over the **landed** subset (5.3.1 carries no flip while a PR is unlanded), and the **end-of-run report** names each PR's end state (landed · parked, awaiting the paired session · skipped, with why); the 5.3.4 sweep-leftovers step must never touch a parked PR's branches (both published sides are what make parked state recomputable).

---

## 5.3 Sprint close — spec sweep, then wrap

Goal: once every PR is processed — run the **one** `spec-writer` sweep over what landed, land it, then inspect coverage and summarise.

### 5.3.1 Spec sweep (the one dispatch)

**One `spec-writer` per refine run, never per PR** (a bounded accuracy or foreign-overlap re-dispatch of that same sweep is not a second sweep). Position the checkout on the landed trunk tip, clean:

```bash
git fetch origin
git checkout --detach origin/development
```

then dispatch **exactly one** `spec-writer` (its agent file carries `isolation: worktree`; native isolation cuts its worktree from `HEAD` — the landed tip). It patches `.spec` for the **whole sprint's landed diff** — the sprint's landed commits on `origin/development` (matched by their subjects + `Ticket:`/`docs(plan)` trailers against the sprint's tickets — identity across the rebased land rides subjects + trailers, so foreign interleaved commits fall out) vs the sprint's base (the trunk commit the sprint forked from at Plan, parent of the landed `docs(plan)`); the union diff is their combined patch. It applies ADDED/MODIFIED/REMOVED hunks in place per `spec-format`, describing the system **as it now IS**.

**Framing — hand it landed-state only.** State the worktree contract (per `spec-writer` / 5.1.3) and give it: the current `.spec`, the sprint's landed union diff, the full content of the touched files, the `src/` listing, `package.json`, the `.adr`, and — where present — the `.sprint` plan, `.brief`, and `.stories` (each skip-if-absent per the agent's own input contract; anything else it self-fetches from its worktree, a full repo copy). **Name no PR, no order, no "earlier/later"** — it describes one current system, so "lands in a later PR" narration has no source.

**Creation mode (no `.spec` yet).** if `.spec/spec.md` is absent (first-ever sprint, or a pre-existing repo adopting the plugin) → `spec-writer` runs in **creation mode**: hand it the **whole codebase** as session-gathered inputs — the full `src/` (or equivalent) listing, the key files the session selects (types, schemas, entry points, config), and the manifest (`package.json` or equivalent) — **not** the sprint diff, and no explorer dispatch (the tool table binds Agent to fix agents and the one sweep). It returns the full nine-section file for the **session** to write to disk (it has Edit, not Write). Never narrow creation to the sprint's slice — the founding spec describes the whole system. The founding commit is a full-file `docs(spec)` — no `sections:` delta line, since there is no prior spec to diff.

**Verify the sweep (autonomous).** Re-derive the spec sections the landed reality touches and assert the delta records exactly them against **reality** — the `src/` listing + `package.json` + the landed diff, not the diff alone: a legitimate drift fix on a section the diff didn't introduce is **expected** (recorded on the commit's `drift:` line, `spec-format`), never rejected as "invented". Assert no forward/PR-relative token leaked. if the check fails → **one** re-dispatch with the discrepancy — **remove the failed sweep's worktree first** (the 5.1.4 tail mechanics), so exactly one sweep worktree ever exists and the collection below has one unambiguous source — then **bounded-then-escalate** (a repeating miss is a signal for the user, not a loop). Pass → land.

**Land the sweep.** **if this run completes the sprint** (every PR landed, none parked/unlanded) → the session first makes the `.sprint` plan `draft → built` flip edit **inside the sweep worktree** — spec and completion become true together in one commit; a sprint that never completes → the flip never lands. Then collect and land — doc-only, no re-verify (the code was verified at 5.1.5):

```bash
git -C "<sweep-worktree>" add -A
git -C "<sweep-worktree>" commit -m "<docs(spec) message per commit-format>"   # the flip rides this commit when the run completes the sprint
git cherry-pick <that commit's SHA>            # main checkout, detached on the landed trunk tip
git push origin HEAD:development               # the FF land; non-FF rejection → fetch · re-position · re-pick · retry
```

**Creation mode lands differently:** the session wrote the founding `.spec/spec.md` in the **main tree** (detached on the landed tip), so there is no sweep worktree to collect — `git add .spec/spec.md` (+ the plan flip when the run completes the sprint) → one full-file `docs(spec)` commit → the same `git push origin HEAD:development`.

After the land, in **both modes**: remove the sweep worktree (`git worktree remove --force` + `git worktree prune`) — absent in creation mode; spent in update mode. The 5.3.4 sweep is only the safety net.

A `.spec` conflict on this land — the trunk push rejecting because a **foreign** sprint touched `.spec` on `origin/development` during the window — is healed with the flow's own halt pattern: fetch, re-position on the fresh trunk tip, re-pick the sweep commit; the pick **halts** on the overlap → dispatch a **general-purpose solo-heal** (`implementer-prompt` Mode B shape), injecting the sweep delta + the foreign landed spec; it resolves the markers → `git add -A && git cherry-pick --continue` → push. (Never re-dispatch the isolated `spec-writer` for this: its frontmatter isolation cuts a fresh worktree that cannot see the halted state.) This is a doc reconcile, **never a park** (parking is for code).

**Parked PR → sweep the landed subset, defer the flip.** if any PR parked this run → the sweep still runs, but over the **landed** subset only: the parked PR's code is not on `origin/development`, so it is out of scope by construction and the spec never describes unlanded code. The commit carries **no flip** (the sprint isn't complete). When the parked PR heals and lands in its paired session, **that** run's 5.3.1 sweep covers the now-complete trunk and carries the flip.

### 5.3.2 Coverage view

Derive `US-### → Issue → PR → .spec`. Did the landed PRs meet the sprint's single Goal (the `.sprint` plan, Plan 1.2.6)? **A gap is a planning miss, not a block** — it stays visible as the open tickets (unbuilt tickets legitimately stay open, Build 3.4.4's invariant) or feeds the next `/sprint-plan`. Inspect, not a gate — landed PRs already shipped.

### 5.3.3 Confirm completion (forge)

All this sprint's PRs landed — the forge **is** the completion record (built tickets were closed at Build 3.4.3; unbuilt ones legitimately stay open). Confirm the plan flip landed on the 5.3.1 spec-sweep commit — **inspect only, no commit ever**. if the plan still reads `draft` → an irregular close (usually a PR still parked): report it — the next sprint's backstop (Plan 1.0.3) catches it; never commit a flip here.

### 5.3.4 Sweep worktrees + branches

Safety net — normally a no-op (Build 3.1.6 removed the t0-writer's worktree, 3.2.6 cleaned per wave, 5.1.4 per fix batch, 5.2.3 per PR, Review 4.6.4 removed its read-surface worktrees, and the 5.3.1 sweep removes its own worktree):

- survivor worker worktrees **and leaked review read-surface worktrees** (`../<repo-dirname>-review-<g>` — the cleanup Review 4.6.4 owns, backstopped here) → `git worktree remove --force <path>` + `git worktree prune`
- leftover landed-PR branches → `git branch -D <name>` + `git push origin --delete <name>` — **never a parked PR's branch** (its published sides are what keep parked state recomputable)
- any completed `[Epic]` issue still open whose child tickets all closed → `gh issue close <epic-N>` (Build 3.4.3 should have — safety net)

### 5.3.5 Present summary

The summary shows landed PRs + the one spec sweep — a run that parked a PR reported that state at 5.2.3 / the end-of-run report:

```
## Sprint Refine — v{N}

PR #<a> <title> — <f> findings fixed · landed
PR #<b> <title> — clean review · landed
Spec: one landed-whole sweep over the sprint diff — landed (docs(spec))

Coverage: US-### → #issue → PR → .spec   (gaps: <open tickets / next-plan input>)
Plan: sprint-v{N} → built (rode the spec sweep)
```

Then recommend the next step — *phases don't share a session; artifacts are the bridge*: the landed trunk, the closed PRs, and the `built` plan carry the state. Next: `/sprint-plan` in a fresh session.

---

## Key principles

- **Parking is not solving** — a conflicted land rebase is aborted (`git rebase --abort`, the branch restored byte-clean), the PR parks for a paired human session, and everything else continues; this skill never auto-resolves a land conflict, and nothing conflicted or half-applied ever survives locally or publishes. Per-PR land conflicts are always code (per PR never touches `.spec`); a `.spec` overlap arises only at the 5.3.1 sweep land (a foreign sprint) and is healed at its halt by a solo general-subagent dispatch — never a parked pair.
- **One spec sweep per sprint** — PRs land code-only; the single worktree-isolated sweep at 5.3.1 sees the landed whole (the design and its lag trade live at 5.0.2; a parked PR sweeps the landed subset and defers the flip to its heal run).
- **Fixes fan out, the land serialises** — each PR's fix-agents run in their own git worktrees (worktree-isolated), so the slow work overlaps across PRs (not one PR at a time), and the session collects each serially onto its PR's branch as sole committer. Peers land any order, a stack lands base-first; the land itself is always serial (one trunk, non-FF rejection as control).
- **Session is the sole author** — fix agents and `spec-writer` are hands, not authors: worktrees collected in, session-authored commits out; workers return status + summary + working directory — never a diff — and never commit, never run git.
- **One atomic commit per fix worker** — `fix(scope): <the file's findings>`; the commit carries every finding the worker owned plus their regression tests; findings from different files are never bundled.
- **Verify is a step, not a push side effect** — 5.1.5 is the explicit gate (Verify + no conflict state), and only then does 5.1.6 publish.
- **One human gate, on content, never vcs ops** — accept the PR against its DoD (5.2.1); the fix-set (5.1.2, default fix-all-published) and the spec sweep (5.3.1, self-verified for accuracy) run autonomously, and every commit, push, and land follows autonomously.
- **The trunk only moves forward** — each PR lands code-only (fetch → rebase the PR branch onto the fetched trunk → publish the rebased head → FF push the trunk; a **stack** base-first: retarget the dependent's base to `development` before deleting the base branch, then rebase + FF), and the 5.3.1 spec sweep lands one doc-only `docs(spec)` commit the same way; a non-FF rejection means fetch · re-rebase · retry; the flow never merges.
- **Flip rides the fact** — the `.sprint` `draft → built` flip is an annotation on the 5.3.1 spec-sweep commit (the run that completes the sprint), made by the session; 5.3.3 only confirms it, never commits.
- **Branch deletion is manual** — rebase-land never fires GitHub's auto-delete; 5.2.3 deletes local + remote.
- **Trust the artifact** — the latest `### Code Review` comment is the work order; never re-review, never re-score, never re-check Tickets/Build decisions.
- **No Haiku, ever** — Sonnet or Opus on every dispatch.
- **Ends at the trunk** — `development → main` promotion is out of scope; this skill spans no other phase.
