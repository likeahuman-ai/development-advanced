---
name: sprint-build
description: "Implements the sprint's approved tickets and opens the sprint's draft PR(s); the history it leaves reads as one atomic commit per ticket, built team-scale by parallel implementers against the spec/ADR knowledge layer, bound for the team's shared development trunk. Use when tickets are ready to build on that trunk/team repo and the user says 'start coding', 'implement this', 'build the tickets', or 'start building' — normally after /sprint-tickets has pinned a build-order; ticket numbers may be passed directly when no pin exists. A solo, sequential, single-PR project belongs to the sibling development plugin instead."
argument-hint: "Ticket numbers (e.g. '#203 #204') — only needed when no pinned build-order exists (degraded path)"
---

# /sprint-build — Build (Phase 3): waves → collect → partition → publish

**Goal: build the approved tickets and open the PR(s).** Implement the build-order's tickets wave by wave with worktree-isolated implementers, collect one atomic commit per ticket onto the integration branch, partition into the divided PR chain(s), publish, open draft PRs, retire the sprint's issues, and hand off to `/sprint-review`.

*Trust the artifact.* The pinned build-order issue from `/sprint-tickets` is the approved plan — `## Parallel Waves` · `## PR Grouping` · `## Scope` · `## Verify` are decided. Execute them as written: never recompute waves, never re-derive the grouping, never re-open the scope. Use the provision command verbatim; `## Verify`'s other entry is the promise `./scripts/t0.sh` — the sprint's gate script, which this phase makes true as its first act (3.1.5–3.1.6). Deviate only when faithful execution would actually break (a referenced issue, file, or symbol does not exist; two instructions directly contradict) — then stop and escalate. Never diverge silently.

Formats referenced as `<name>-format` live at `skills/sprint-build/formats/<name>-format.md`; prompts referenced as `<name>-prompt` at `skills/sprint-build/prompts/<name>-prompt.md` (under `${CLAUDE_PLUGIN_ROOT}`) — the filename keeps the `-format`/`-prompt` suffix.

**No human gates in this phase.** Build runs autonomously from the build-order to the draft PRs; its gates are agent/verify checks, which ride inline (they are not their own named steps). Pause only to escalate.

**The session is the sole author of history** (*subagents are hands, not authors*). Implementers never commit and never run git — they write files, self-verify, and return their status, a summary, and their working directory. The session commits each worker's tree, cherry-picks every ticket onto the integration branch, owns every git/gh motion, and writes no code itself.

**Initial request:** $ARGUMENTS

---

## 3.1 Phase setup

**Goal:** re-assert the branch position, check preconditions, find the build-order + read its tickets — and make the `## Verify` promise true: the sprint's gate script generated, proven, and collected as the founding commit before any wave.

### 3.1.1 Re-assert the position

Identify the sprint from the latest `.sprint/sprint-v{N}.md` draft, then confirm the checkout sits on the sprint's integration branch with the plan on it:

```bash
git branch --show-current                                        # expect feat/sprint-v{N}
git log --oneline origin/development..HEAD --grep='docs(plan)'   # the v{N} plan commit must appear
```

A match for the v{N} `docs(plan)` commit must exist in the branch's ahead-of-trunk range.

- if the branch does not exist, or no commit in `origin/development..HEAD` is the v{N} `docs(plan)` → **stop** — the plan never crossed onto the integration branch (Plan 1.3.2 never ran): route back to `/sprint-plan`. Never silently cut a fresh branch.
- The checkout is shared mutable state — another session can move it. if `git status --porcelain` shows uncommitted state this session did not create → surface it, don't blind-switch over another session's work.

### 3.1.2 Check worktree preconditions

One-time repo facts, checked at point of need — Build breaks without them:

- **Isolation is native — no plugin hooks.** The writing agents' `isolation: worktree` (the `t0-writer` and `implementer` files) makes Claude Code cut each worker its own linked git worktree of this repository, based on the session's `HEAD`. Basing is therefore a session discipline, not a hook's: at every dispatch, **`HEAD` sits on the integration tip** (the branch tip — the last collected commit) **and the tree is clean** (`git status --porcelain` empty). The collection discipline (3.2.4 ends every wave fully committed) guarantees both; confirm them before **every dispatch** — the 3.1.5 founding dispatch, a wave's opening batch, and each 3.2.3 re-dispatch alike — and **record the confirmed `HEAD` as that dispatch's base** (the referent its collection asserts against). A dirty tree or mispositioned `HEAD` would base the worker on the wrong state.
- **The survivor gate — no `scripts/t0.sh` may pre-exist at generation.** The gate script is sprint-local, disposable infrastructure; the `t0-writer` derives it from the repo's tooling alone, and that guarantee is structural: before the 3.1.5 dispatch fires, assert the path does not exist on the tree. **The test is history-keyed, never existence alone:** if `scripts/t0.sh` exists, look for this sprint's founding `chore(t0)` commit in `origin/development..HEAD` — **present → the promise is already true: skip 3.1.5–3.1.6 and continue at 3.2** (a re-entered Build never deletes or regenerates its own proven script); **absent → a stale survivor from a predecessor's unclosed sprint** — a repairable precondition: delete it (the repair commit below), so the agent's worktree, cut from the session's `HEAD`, is born scriptless and a predecessor's text can never anchor the generation.
- **Build-needed gitignored config (`.env`, secrets the gate needs).** A git worktree checks out only *tracked* files. Derive the needed-file list once here, from the build-order: the gitignored paths its **provision** command and the gate script's checkers read (`.env` files, local config the tooling references — check each against `git check-ignore`). if the needs aren't derivable → ask the user once for the gitignored paths the gate needs, and record the answer for this run. Most repos have none — an empty list is a valid result, stated in each brief. The session lists it in each brief (3.2.1) and the worker copies the files in from the main checkout before provisioning — plain file copies, never git.

if a precondition fails a check the session can repair in-repo — the worktree parent dir (`../`-sibling paths) is not excluded by the committed `.gitignore`, a config file the list marks copy-needed should instead be tracked, or a stale `scripts/t0.sh` survives from a predecessor's unclosed sprint — no `chore(t0)` in `origin/development..HEAD` (delete it) — → set it before any dispatch (one-time repo setup, not a deviation) — then commit the fix as its own session-authored commit, the trailer inside the quoted message:

```bash
git add <the fixed paths>
git commit -m "chore(repo): worktree isolation preconditions

Assisted-by: session <model>"
```

so the fix rides the chain instead of surviving as dirty state, and the tree returns to clean before the first dispatch.

### 3.1.3 Find the build-order issue

Find the pinned build-order issue — **authoritative**: `## Parallel Waves` · `## PR Grouping` · `## Scope` · `## Verify` (the provision command + the gate promise `./scripts/t0.sh`). Trust it; never recompute.

- if no pin → **degraded path**: identify the tickets from `$ARGUMENTS`, generate the order in-session, and **flag it** — the order normally exists; a missing pin is a signal, not routine.

### 3.1.4 Read the listed tickets

Read the full issue bodies the build-order references by `#`:

```bash
gh issue view <N> --json number,title,body,labels
```

Objective, requirements, AC, `.spec`/`.adr` pointers — the raw material for the implementer briefs (3.2.1).

### 3.1.5 Dispatch the t0-writer — Build's first act

The gate script the build-order promises does not exist yet; this dispatch makes the promise true. Dispatch the **`t0-writer` custom subagent** — the phase's **first dispatch**, launched as soon as 3.1.4's ticket reading completes (the footprint below derives from the ticket bodies, so the dispatch waits for them — and for nothing else) and running in parallel with the remaining setup (3.2.1's wave-1 briefs). Its worktree isolation, effort, and fixed Opus tier all ride its agent file — never per-call flags, and never a per-dispatch tier pick.

**Hard dispatch precondition — the survivor gate (3.1.2):** `scripts/t0.sh` must not exist on the tree when this dispatch fires. The agent's worktree is cut from the session's `HEAD`; a scriptless tree is what makes its generation derive from the repo's tooling alone. (A re-entered Build whose founding `chore(t0)` commit already sits on the chain never reaches this dispatch — 3.1.2 routes it straight to 3.2.)

The brief is thin — three inline facts; the agent file owns the whole method:

- the **sprint footprint** — every package the sprint's tickets touch. Derivation is fixed, not discretionary: the set of packages containing any in-scope ticket's `## Context` anchor paths, plus any package a ticket's `new:` created-path anchors introduce (the build-order carries no per-ticket file attribution by design — the ticket bodies' `## Context` anchors are the source). State the resulting package list in the brief;
- the **provision command** — from `## Verify`, verbatim;
- the **return contract** — the closed status vocabulary (SUCCESS | NEEDS_CONTEXT | BLOCKED); the report shape is the agent file's own.

- if `NEEDS_CONTEXT` / `BLOCKED` → fill the named gap and re-dispatch — bounded (two rounds), then escalate. A BLOCKED naming a contradiction with the build-order (a provision command that fails outright, a footprint package that does not exist) is a Tickets signal — escalate, never improvise a substitute.

### 3.1.6 Collect the founding commit — no wave before the script

Barrier on the t0-writer **before any wave dispatches** — 3.2.1's brief-writing may continue meanwhile; 3.2.2 may not fire. Per its SUCCESS report: verify isolation engaged (3.2.3's fail-loud check, against the base recorded at the 3.1.5 dispatch), then assert **`scripts/t0.sh` is the only changed path in the worker's tree** — any other surviving diff (a leftover proof seed) fails the collection: re-dispatch with the named defect, never collect it. Then collect — the session's own hands, the 3.2.4 mechanics:

```bash
git -C "<worker-dir>" status --porcelain     # exactly one path: scripts/t0.sh — anything else fails the collection
git -C "<worker-dir>" add scripts/t0.sh
git -C "<worker-dir>" commit -m "chore(t0): generate and prove the sprint gate script

Assisted-by: t0-writer <model>"
git cherry-pick <that commit's SHA>          # main checkout, HEAD on the integration tip
```

Then remove the worker's worktree — 3.2.6's mechanics (`git worktree remove --force "<worker-dir>"` + `git worktree prune`): the founding commit lives on the chain; the worktree is spent.

**The founding commit is the wave law: no implementer wave dispatches before this commit exists on the integration branch.** Every worker worktree thereafter is born carrying the proven script, and the proof run's provision was the sprint's first cold install — it warmed the shared store for every subsequent worktree.

**Frozen for the sprint, upward-only amendment.** From this commit on, the script is untouchable by every agent it checks — an agent's edit to `scripts/t0.sh` surfaces as a diff on a file its work order doesn't own and fails at collection. The **session** may amend it strictly **upward** — scope may grow, dials may rise, never the reverse (widening can't be gaming) — and the amendment has exactly one shape: **trigger** — a gate red on legitimately in-scope work, or a footprint the sprint outgrew; **act** — re-dispatch `t0-writer` in amend mode (the one sanctioned re-dispatch: the brief names the standing script as the floor and the added scope or raised dials; the agent re-proves every addition with its own fresh seeded error, closing the full green-red-green protocol); **collect** — this step's mechanics, as `chore(t0): widen the sprint gate`. The session never edits the script by hand — *subagents are hands, not authors* holds for infrastructure too. **A published review finding that targets `scripts/t0.sh` routes here — to the session as an upward amendment — never to a fix worker:** a fix agent editing the oracle it must pass is the gaming the freeze exists to bar. Removal is not this sprint's act: the script lands with the sprint, and the **next** Build's survivor gate (3.1.2) deletes it, so the next generation starts blind.

---

## 3.2 Execute — the wave loop

**Goal:** implement the tickets wave by wave, verified. **Loop 3.2.1–3.2.7 per wave until all waves are built, then → 3.3.**

| Step | Act | Output |
|---|---|---|
| 3.2.1 | prepare one `implementer-prompt` brief per ticket | dispatch briefs |
| 3.2.2 | dispatch the `implementer` subagents (isolation in their yaml) | one worktree per ticket, cut at the wave base |
| 3.2.3 | barrier — collect statuses; verify isolation engaged | worker trees verified, ready to collect |
| 3.2.4 | collect one atomic commit per ticket (commit in the worktree → cherry-pick), in build-order | the chain grows |
| 3.2.5 | the gate — spec-review + no conflict state + chain asserts | certified wave tip |
| 3.2.6 | remove the wave's worktrees + assert chain integrity | clean tree, chain intact |
| 3.2.7 | no push — loop or finalize | waves remain → 3.2.1 · done → 3.3 |

### 3.2.1 Prepare dispatch prompts

Write each implementer's brief from `implementer-prompt` — *direction from the session, discretion to the agent*: inject only the cheap context the session already holds; the agent expands it at its own discretion. The session hands in:

- the **full ticket body** (3.1.4)
- the ticket's **`.spec` slice** (from `.spec/spec.md` — skip if absent)
- the governing **`.adr` Y-statement** (skip if absent or the ticket names no ADR)
- the **coding-standards pointer** (if installed)
- the build-order's **provision command**, verbatim — the gate invocation `./scripts/t0.sh` is fixed in the prompt itself (the script sits in the worker's tree: the founding commit, 3.1.6, predates every wave)
- the **build-needed config list** (3.1.2 — absolute source paths in the main checkout; empty in most repos)

The prompt owns the worktree contract — reference it, never restate it; the collection (3.2.3–3.2.4) relies on the invariants stated there once, at the dispatch surface.

### 3.2.2 Dispatch implementers (worktree-isolated)

Record the **wave base** first — `git rev-parse HEAD` (the integration tip; 3.1.2 just confirmed the tree is clean on it). Every worker's worktree is cut from it, every worker in the wave is parented on it, and 3.2.4 collects them into a linear chain on top of it.

Dispatch one `implementer` per ticket in the wave — the **`implementer` custom subagent, briefed with `implementer-prompt` (Mode A)** — fanning the wave's independent tickets out in one parallel batch. **Isolation is not passed here: the `implementer` agent file carries `isolation: worktree` (and `effort`) in its frontmatter, so every spawn is worktree-isolated deterministically — never a per-call flag the session can drop.** Every dispatch still resolves a **pinned capable tier from the ticket's size** (`S`/`M` → sonnet, `L` → opus — Sonnet or Opus, never Haiku; no silent inherit) and passes it as the per-dispatch `model` override (it beats the agent file's floor): the commit's `Assisted-by:` trailer (3.2.4 / `commit-format`) IS the per-dispatch tier record, so every implementer commit shows the Capable tier it ran on.

- Dispatch the wave **as written** in `## Parallel Waves` — a hard-dep wave, nothing to recompute. Each implementer writes its own worktree → no clobber.

### 3.2.3 Collect results & verify isolation

Wait for the wave (barrier); collect each implementer's **status + summary + working directory**. There is no diff — the changes live on disk in each worker's worktree.

**Verify isolation engaged before any collection — fail-loud (the PF-11 class).** Per `SUCCESS` worker:

```bash
git -C "<worker-dir>" rev-parse --git-common-dir   # resolves into THIS repo's .git — a linked worktree, not a stranger clone
git -C "<worker-dir>" rev-parse HEAD               # equals the wave base recorded at 3.2.2
```

and the reported working directory is never the repo root. A silent no-op isolation — a worker reporting the main checkout as its directory — means its edits are commingled with the session's tree: surface the failure, then discard the commingled edits — the tree was asserted clean at dispatch (3.1.2), so every path `git status --porcelain` now lists is that worker's; confirm the list against the worker's reported summary, then `git restore .` (tracked modifications) **and** `git clean -fd -- <those listed paths>` (its new untracked files — pathspec-scoped, never a bare `git clean`), and re-dispatch. A wrong base → discard the worktree, re-dispatch. Never rationalise either as harmless.

- if `NEEDS_CONTEXT` / `BLOCKED` → re-dispatch with the gap filled — **bounded (two rounds), then escalate** (3.2.5's pattern).
- A failed / blocked / verify-fail worker is **never collected** — its worktree is removed unmerged at 3.2.6.

### 3.2.4 Collect the wave's commits

The session collects **one atomic commit per ticket, in build-order** — the sole author. Two acts per ticket: commit the worker's tree inside its worktree, then cherry-pick that commit onto the integration branch. No textual patch, no `git apply` — the worker's changes become a commit exactly once, under the session's hands.

**Happy path** — per ticket, in build-order:

```bash
git -C "<worker-dir>" status --porcelain -- scripts/t0.sh   # EMPTY — any hit is a freeze violation (3.1.6): fail the collection, re-dispatch with the named defect, never collect
git -C "<worker-dir>" add -A                    # tracked edits + new files; ignored build debris never enters
git -C "<worker-dir>" commit -m "<message>"     # worker-suggested subject, session-owned, + trailers — commit-format
git cherry-pick <that commit's SHA>             # main checkout, HEAD on the integration tip — appends the ticket
```

The first line is the freeze made mechanical: the script belongs to no ticket, so a worker tree that touched it never collects — the wave's one untouchable file is asserted per ticket, before anything stages.

Each pick lands the ticket as the chain's next commit, so the wave lands as a linear, one-commit-per-ticket chain in build-order. (The worker's original commit stays behind in its worktree and dies at teardown, 3.2.6 — the chain's cherry-picked commit is the one that lives.)

**Same-line overlap (rare)** — picking a ticket whose change touches the same lines as an earlier one **halts at a controlled halt**: conflict markers in the tree, the operation suspended, nothing committed (*conflicts never reach the trunk — and never block the queue*: conflicted or half-applied state never publishes; the halt is bounded and session-owned):

1. Never commit markers, and never abort the pick to skip or re-serialise the ticket — the halt is exactly where the conflict heals; aborting hides a real coupling.
2. Dispatch a **general-purpose solo-heal subagent** (an `implementer-prompt` **Mode B** dispatch) on the halted tree: resolve the conflict markers in place so both intents stand — main tree, no worktree, no provision, no returned diff. **Mode B is general-purpose by design — never the `implementer`/`fix-agent` custom subagents, whose frontmatter would force a worktree onto what must be a main-tree in-place heal.**
3. The session continues the operation: `git add -A && git cherry-pick --continue` — the ticket's message and trailers ride unchanged.
4. **Two heal rounds per conflict, then escalate.** Log the overlap as a **coupling signal**: name the colliding ticket pair and file on the PR body's `### Wave Interactions` line (3.4.2) — it feeds the next decomposition.

**Dep-change ticket** — the worker ran provision **inside its worktree** (per its brief), so a regenerated lockfile is already part of its commit and picks like any other change. A lockfile that conflicts at a pick is **never line-merged**: keep the picked ticket's manifest, delete the conflicted lockfile, re-run the provision command at the halt to regenerate it, then continue.

**Wave-end assert** — after the last ticket's pick, close the collection by asserting the wave ended as intended:

```bash
git log --oneline <wave-base>..HEAD         # exactly this wave's tickets, one commit each, in build-order
git log --merges origin/development..HEAD   # empty — the wave ends linear: trunk tip → finished tickets, no merge commits
git status --porcelain                      # empty — no halt left open
```

**No session verify run closes the wave** — that site is deliberately removed. The wave's machine acceptance is each worker's own green `./scripts/t0.sh` run, already made before it reported (only green workers were collected, 3.2.3); a within-wave sibling collision that slips past the picks surfaces at the next wave's agent runs or at Refine's standing pre-land gate, by design. The gate (3.2.5, immediately next) judges these asserts and completes the remaining checks — a wave never loops back to 3.2.1 ungated.

### 3.2.5 Post-integration checks — the gate

The gate's checks, riding inline rather than as their own named steps:

1. **Spec-review** — per built ticket, dispatch a read-only reviewer (a **general-purpose subagent** — no agent file; `spec-review-prompt` is its complete brief) with the ticket body (its AC) + its `.spec` slice + the ticket's collected commit; one parallel batch. It judges: built *what was asked*, not just "passes". if no `.spec` → the reviewer judges against the ticket's AC alone.
2. **No conflict state remains** — the tree is clean and no operation is suspended:

```bash
git status --porcelain      # empty — no unmerged paths, no cherry-pick in progress
```

3. **The chain asserts hold** — the wave-end range (3.2.4) read linear, one commit per ticket, no merges.

**No machine-verify runs at this gate** — the removed site, deliberately: the sprint's one in-loop check is the proven gate script, and it runs where the work is — every worker green in its own worktree before reporting, and Refine's standing gate before the land. Nothing else pretends to gate.

Branches:

- if **spec-review fails** → **escalate, never auto-fix** — built-the-wrong-thing is a ticket/decomposition signal; an agent guessing the intended behaviour compounds the miss.
- if **conflict state remains** → a halt was left open — return to 3.2.4's halt discipline (heal → continue); bounded (two rounds), then escalate.
- if **a chain assert fails** → STOP and escalate — collection corrupted the chain (3.2.6's integrity framing; never rationalise).
- PASS (all three) → 3.2.6.

### 3.2.6 Remove the wave's worktrees + assert chain integrity

After the gate passes — per worker worktree. The harness auto-cleans a worktree it cut only when the worktree is unchanged, so the session runs teardown **explicitly** as the load-bearing cleanup — idempotent either way:

```bash
worker_branch=$(git -C "<worker-dir>" rev-parse --abbrev-ref HEAD)   # read BEFORE removal — the worktree is the only place the name is readable; "HEAD" = detached, nothing to delete
git worktree remove --force "<worker-dir>"   # ignored provision debris may sit in the tree; the ticket's commit is already on the chain
git worktree prune
git branch -D "$worker_branch"               # only if the read gave a real branch (not "HEAD") AND it is neither the integration branch nor a PR branch — its commit lives on as the chain's cherry-pick
```

A `SUCCESS` worker's change was cherry-picked onto the chain (3.2.4), so removing its worktree loses nothing; a discarded/blocked worker's tree was never collected and dies with the worktree, by design.

**Post-cleanup chain-integrity assert.** Confirm the founding `docs(brief)`/`docs(plan)` commits and every prior wave's commit still sit on the chain, in order, with nothing foreign:

```bash
git log --oneline origin/development..HEAD   # founding docs + every collected ticket, in build-order — nothing missing, nothing extra
git log --merges origin/development..HEAD    # empty — the flow authors no merge commit, ever
git status --porcelain                       # empty — clean tree
```

if a founding or prior-wave commit is missing, or the range shows a merge or a stray commit → **STOP and escalate**: isolation or collection corrupted the chain (a regression — never rationalise it as intact). On a healthy run the chain reads trunk tip → founding docs → all collected tickets → clean tree.

### 3.2.7 Loop or finalize

**No per-wave push** — durability is local; the integration branch accumulates commits in place. The first push is publication (3.4.1).

- if waves remain → next wave (3.2.1)
- else → 3.3

---

## 3.3 Finalize

**Goal:** partition into the divided PR chain(s) from the verified integration tip.

### 3.3.1 Partition into PR chain(s)

From the build-order's `## PR Grouping` (coupling + dependency-closed — decided at Tickets 2.5.3, trust it). Each group is marked **peer** (depends only on what's already on `development`) or **stacked** (hard-depends on a prior group *in this sprint*) — read that mark, never recompute it.

**One group → no surgery.** The integration branch *is* the PR branch → 3.4.

**>1 group → rebuild each group by cherry-pick.** Build integrated **by wave** (3.2), so the chain is wave-ordered and a group's commits are interleaved with other groups' — non-contiguous. No reorder pass exists: each group's branch is **rebuilt directly** — cut the group's base, cherry-pick its commits in chain order. Map each chain commit to its group by its `Ticket:` trailer against the group's ticket list (`git log --reverse --format='%H %s' origin/development..<chain-tip>`, trailers via `git log -1 --format=%B <sha>`). Work base→dependent — a stacked group after the group it depends on; peers in any stable order:

- **Peer group** — branch at the trunk, pick the group's commits in chain order:

```bash
git checkout -b feat/sprint-v{N}-<g> origin/development
git cherry-pick <sha> <sha> …                # the group's commits, chain order
```

  In an **all-peer** sprint the **first** group (in `## PR Grouping` order) picks the founding `docs(brief)`/`docs(plan)` commits first, at its base — the sprint's documentation lands with a designated PR, never orphaned; later groups pick ticket commits only. In a **mixed** sprint (any stacked group) the founding docs ride the **first** stacked group's base (in `## PR Grouping` order — the same tiebreak as the all-peer case) instead, and **every** peer picks ticket commits only — the docs publish exactly once.

- **The gate script rides every branch — unlike the docs, deliberately.** Every **peer** group picks the script's `chore(t0)` commits — the founding commit and any upward amendments (3.1.6), in chain order — **at its base, before its ticket commits** (in a group that also carries the founding docs, chain order puts the docs first); a **stacked** group inherits them through its base and never re-picks them. Every branch that will ever run a gate — each PR tip, every fix worktree cut from a PR head, Refine's standing pre-land gate — is thereby born carrying the proven script. The docs keep their publish-once rule; the script deliberately doesn't: at land the duplicate drops by **patch identity** (a rebase onto a trunk already carrying the identical patch skips it), so the trunk receives the script exactly once, from whichever PR lands first.

- **A clean pick is the dependency-closure signal.** if a peer group's pick halts on a conflict → the group wasn't dependency-closed → **flag, don't force**: `git cherry-pick --abort`, name the grouping miss (a Tickets 2.5.3 signal), never resolve a peer group into existence. This abort is legitimate — it discards only the misgrouped branch attempt; the integration chain still holds everything.

- **Stacked group** — branch at the prior group's tip, pick its commits:

```bash
git checkout -b feat/sprint-v{N}-<g> <base-group-tip>
git cherry-pick <sha> <sha> …
```

  Its base carries everything it depends on, so its picks apply clean — that is what stacking buys. if a stacked pick halts → the grouping is wrong somewhere (a dependency outside its declared base) → abort and flag, never force.

- **No per-tip verify runs here — the removed site, deliberately.** A clean pick is the dependency-closure signal; the machine gate already ran where the work was (every collected worker's own green script run), and runs again at Refine's standing pre-land gate on each PR before it can land. A group that applies clean yet hides a cross-group semantic break surfaces there — the recorded trade of running the gate at exactly two sites.

- Once every group's branch is rebuilt — every pick clean — → **delete the consumed integration branch**: `git branch -D feat/sprint-v{N}` — its commits live on as the groups' cherry-picks; nothing dangling. Cherry-pick re-hashes commits — identity across the rebuild rides subjects + trailers, never SHAs.

---

## 3.4 Phase close

**Goal:** publish the branch(es), open the PR(s), retire the forge objects, hand off to review.

### 3.4.1 Push the branch(es)

**Publication** — per PR branch, in build-order (a stack's base group first):

```bash
git push -u origin feat/sprint-v{N}(-<g>)
```

`<g>` = the group's slug from `## PR Grouping` — deterministic, two runs name alike; one group → no `-<g>`. Nothing was pushed before — durability was local; the branch names existed locally, and publication creates the remote refs. For a stack, pushing the base and each dependent publishes **one linear history under several refs** — no duplication, no force. **Push is transport, never a gate — and this publication is consciously ungated:** the branches carry only collected state whose workers reported the gate script green, plus clean picks at partition; commits are private, pushes are public, and the guard sits on the one push that matters — Refine's standing pre-land gate runs the script before anything reaches the trunk.

### 3.4.2 Create the PR(s)

One **draft** PR per PR branch, body per `pr-format`. Filling `### Wave Interactions`: read forward every same-line overlap 3.2.4 logged at a collection halt — each is a collection-overlap line beside the cross-wave/intra-wave contracts. The `--base` follows the group's mark (3.3.1):

```bash
# peer group, or a stack's base group → based on development
gh pr create --draft --base development --head feat/sprint-v{N}(-<g>) --label needs-review --title "..." --body-file <pr-body>
# a stack's dependent group → based on the prior group's branch
gh pr create --draft --base feat/sprint-v{N}-<prior-g> --head feat/sprint-v{N}-<g> --label needs-review --title "..." --body-file <pr-body>
```

A stacked PR's `--base` is the prior group's branch — so GitHub diffs it against that base, showing only the dependent group's delta, and Refine lands the stacked chain base-first (5.2.2). **Draft blocks the UI merge buttons** — the machine guard against the catastrophic accident: a squash-merge click collapsing the atomic chain. Review lifts draft at 4.1.3. `needs-review` is the flow state; draft is the machine guard — apply both.

### 3.4.3 Retire the sprint's issues

**Closed = built.** Tickets are Build's work orders, as review comments are Refine's; acceptance + completion live on the PR — the link in each closed issue carries the trail (an abandoned PR → reopen by hand, exceptional).

```bash
gh issue close <N> --comment "Built in <PR URL>"
gh issue unpin <build-order-N>
gh issue close <build-order-N> --comment "Build complete. PR(s): <URLs>"
gh issue close <epic-N> --comment "Epic complete — all child tickets built."   # ONLY epics whose children are ALL now closed
```

- Close every **built** ticket issue with its PR link.
- Unpin + close the build-order issue with the PR link(s) — the pin slot comes back (Tickets 2.5.7).
- **Close completed epics.** After the ticket + build-order closes, close each `[Epic]` issue whose child tickets (the issues threaded `Part of #<epic>`, Tickets 2.4.4) are **all** now closed — *close-when-last-child-closes*, so an epic spanning sprints with unbuilt children stays open. Without this, shipped epics linger open forever and `gh issue list --state open` stops reflecting work actually in progress.

### 3.4.4 Handoff

Summarise what was built — the PR URL(s) + their groups — and what wasn't: **unbuilt/descoped tickets simply stay open** — the open issue is the durable not-built record (Refine 5.3.2 reads it). Recommend running `/sprint-review` in a **fresh session** — the PR(s) + `needs-review` carry the handoff state.

---

## Key principles

- **Orchestrate, never implement** — every write is a dispatched subagent (the founding `t0-writer`, Mode A implementers, plus a general-purpose solo-heal at a halted pick); the session authors history, not code (*subagents are hands, not authors*).
- **One proven gate, run where the work is** — the sprint's whole in-loop check is the founding `scripts/t0.sh`: generated and proven by `t0-writer` before any wave, frozen for the sprint (session amendments upward only), run by every worker to green before it reports. No session re-run pretends to gate.
- **The session collects every commit** — commit the worker's tree inside its worktree → cherry-pick onto the integration branch; one atomic commit per ticket, in build-order. Workers never run git and return no diff — there is no worker branch or patch to merge.
- **Trust the build-order** (*trust the artifact*) — waves, grouping, scope, provision, Verify are decided upstream; never recompute. if faithful execution would actually break → stop and escalate; never diverge silently.
- **Conflicts heal at a controlled halt** (*conflicts never reach the trunk — and never block the queue*) — a halted cherry-pick holds the conflict at the point of integration; a **general-purpose Mode B solo-heal** resolves the markers in place (never the `implementer`/`fix-agent` custom subagents — their frontmatter would force a worktree onto an in-place heal), the session continues the operation; conflicted or half-applied state never publishes, and message + trailers are never in play. Two heal rounds per conflict, then escalate.
- **Push is publication** — not durability (local commits carry that) and not a gate (the publication is consciously ungated; the machine gate ran in every worker's worktree and runs again at Refine's pre-land gate). Nothing pushes before 3.4.1; push only finished, collected state.
- **Provision unconditionally** — a fresh worktree is write-isolated, not build-isolated; it ships no project dependencies.
- **Bounded, then escalate** — re-dispatch with the gap filled, two rounds at most; a repeating failure is a signal. Spec-review failure escalates immediately — never auto-fix.
- **Capability over cost** — every dispatch runs on a capable tier: Sonnet or Opus, never Haiku.
- **Stay in phase** — `/sprint-tickets` produced the build-order; `/sprint-review` consumes the PRs. Hand off at the boundaries; never execute their work.
