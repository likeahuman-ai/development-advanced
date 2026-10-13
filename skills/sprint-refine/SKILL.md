---
name: sprint-refine
description: "Fixes the Fix findings of each reviewed ticket PR, keeps each ticket one commit, brings every stack up to date, and marks the PRs ready for a person to merge. Runs from start to end without a person. For a team that builds in parallel stacked PRs; a solo, single-PR project uses the development plugin. Never merges: a person merges each stack, and the next /sprint-plan closes the sprint."
argument-hint: "PR numbers (optional; default: every PR labelled needs-refine)"
disable-model-invocation: true
---

# Sprint refine

Fix what the review published, and hand the stacks to a person to merge. The latest `### Code Review` comment of each PR is its work order: never re-review, never reopen the ticket. Fix every Fix finding; leave every Note for the person.

Run to the end without a person. The PR list is your checklist: keep it current as each PR is fixed, restacked, ready or stopped. Stop only when nothing can move without the user. These are not reasons to stop: a summary that announces the next step, an offer to carry on, a list of choices none of which blocks you, or a sense that the turn is long or a milestone is done. Put progress updates in the same message as the next action. Wait for running agents and commands, and collect their results before you treat their work as finished. Read a format again before you write its artifact if it is no longer in your context. If one PR is blocked, continue the others. The run ends when every PR is ready, or stopped with a reason.

**Initial request:** $ARGUMENTS

## Start

Start is your own work.

1. Run `gh auth status`. If the user is not authenticated, tell the user to run `gh auth login`, and stop.
2. Check that `.claude/settings.json` or `.claude/settings.local.json` sets `"worktree": {"baseRef": "head"}` and `.gitignore` lists `.claude/worktrees/`. If not, report what to add, and stop.
3. Run `git status --porcelain`. If the tree is not clean, apart from `.claude/worktrees/`, stop and report the dirty paths. Run `git fetch origin`.
4. Read `finding-format` (`${CLAUDE_PLUGIN_ROOT}/skills/sprint-review/formats/`), and `commit-format` and `pr-format` (`${CLAUDE_PLUGIN_ROOT}/skills/sprint-build/formats/`). They are the source of truth. If this skill and a format disagree, the format wins.
5. Read the `## Verify` section of the sprint's build-order issue: the provision and check commands. List the gitignored files they read, as Build does.
6. Find the PRs: the numbers in the request, or `gh pr list --label needs-refine --limit 500 --json number,headRefName,baseRefName,headRefOid`. Record each PR's head SHA: Restack needs it. For each, read its latest `### Code Review` comment and its Fix findings.

## Fix

A PR with no Fix findings needs no fix: go to Ready.

### Dispatch

A worktree starts from the checkout's `HEAD`. For each PR with Fix findings: `git switch --detach origin/<headRefName>`, check that the tree is clean apart from `.claude/worktrees/`, then send one `fix-agent` with its filled prompt. The PRs fix in parallel.

### Collect

When a fix agent returns:

1. If it reports `failed`, read why. If the cause is the attempt, not the findings, send the PR once more to `fix-agent-opus`, with the first report and the first worktree; keep that worktree until the retry is collected. If the retry fails too, stop that PR and report it.
2. Check the worktree as Build's Collect step 2 does: a linked worktree of this repository, never the main checkout.
3. Read the diff of every test file in the worktree. A test may change only where a finding says it does not prove its criterion; new regression tests are expected. If a test is weakened, skipped or deleted otherwise, do not collect: dispatch again once and name the test. If the second attempt does the same, stop the PR.
4. Amend the ticket's one commit with the fix, and add the fixer's line: `git -C <dir> add -A`, then `git -C <dir> commit --amend --no-edit --trailer "Assisted-by: <agent> <model> <effort>"`. The ticket stays one commit. Move the PR's branch to it: `git branch -f <headRefName> <sha>`.
5. Update the PR body per `pr-format`: the Evidence from the fix agent's report, and the Summary and Merge Danger when the fix changed them.
6. Remove the worktree.

## Restack

When every PR of a stack is collected or stopped, bring the layers above each fix up to date. A stopped PR ends refine for the layers above it: restack and ready only the layers below it, and report the rest.

Work from the bottom of the stack up. For each layer whose branch below it moved, rebase only its own commit onto the new tip below: `git rebase --onto <new tip below> <old head below> <branch>`, where the old head is the SHA recorded at Start. Then `git switch --detach <branch>`, so no branch stays checked out.

If a rebase conflicts, resolve it yourself, as Build's Collect step 6 says: keep both intents, never commit a conflict marker, and never abort to skip the conflict. If you cannot resolve it, stop that stack and report it.

Run the provision and check commands on every layer that changed. Push each changed branch: `git push --force-with-lease origin <branch>`.

## Ready

For every PR whose stack is up to date and green: `gh pr ready <N>`, then `gh pr edit <N> --remove-label needs-refine`.

## Stop

Show the result:

- per stack: each PR, with its ticket, its link and its state
- every finding a fix agent judged mistaken, with its reason
- per PR, the Note findings, for the person who merges
- every PR or stack that stopped, with the reason

Name the next step: a person reads each stack and merges it, bottom first, with `gh stack merge --rebase` or in GitHub. When every ticket of the sprint is merged, the next `/sprint-plan` closes the sprint and brings `.spec` up to date. Never merge.

## Subagents

| Agent | Prompt | Use it for |
|---|---|---|
| `fix-agent` | `${CLAUDE_PLUGIN_ROOT}/skills/sprint-refine/prompts/fix-prompt.md` | fixing one PR's Fix findings in its own worktree |
| `fix-agent-opus` | `${CLAUDE_PLUGIN_ROOT}/skills/sprint-refine/prompts/fix-prompt.md`, with the first attempt filled | fixing them again, on Opus, after the fix agent failed |

Use only the agents in this table.

Read the agent's prompt, fill its slots, and send the filled prompt as the `prompt` of the Agent call. It is the whole task: the subagent sees nothing of this conversation. Send pointers, not copies: the ticket and PR numbers, the findings and the commands. Never pass `name` on the Agent call.
