---
name: sprint-build
description: "Builds the sprint's tickets into stacked draft pull requests: one ticket, one commit, one PR. Builds as many tickets in parallel as their blockers allow, each implementer in its own worktree; the session commits, stacks and opens the PRs. Runs from start to end without a person. For a team that builds in parallel stacked PRs; a solo, single-PR project uses the development plugin. Never plans, never merges."
argument-hint: "Ticket numbers (optional; default: every open ticket of the sprint)"
disable-model-invocation: true
---

# Sprint build

Build tickets that are already decided. Never reopen the plan. There is no interview, no clarifying round, no proposal of a different approach. The person who typed the command expects draft PRs on GitHub when it ends.

If a ticket cannot be built as written, because a file it names does not exist or two of its instructions contradict, stop that ticket and report it. That is a signal for `/sprint-plan`, not a reason to improvise.

**Initial request:** $ARGUMENTS

## Start

Start is your own work.

1. Run `gh auth status`. If the user is not authenticated, tell the user to run `gh auth login`, and stop. Run `gh --version`. If it is older than 2.94.0, tell the user to upgrade `gh`, and stop: blocker links need it. Run `gh extension list`. If `gh stack` is missing, tell the user to run `gh extension install github/gh-stack`, and stop.
2. Check that `.claude/settings.json` or `.claude/settings.local.json` sets `"worktree": {"baseRef": "head"}` and `.gitignore` lists `.claude/worktrees/`. If not, report what to add, and stop: without them, an implementer starts from `main` instead of its blocker, or the worktrees dirty the tree.
3. Run `git status --porcelain`. If the tree is not clean, apart from `.claude/worktrees/`, stop and report the dirty paths. The worktrees start from this checkout. Run `git fetch origin`.
4. Read `ticket-format` (`${CLAUDE_PLUGIN_ROOT}/skills/sprint-tickets/formats/`), and `commit-format` and `pr-format` (`${CLAUDE_PLUGIN_ROOT}/skills/sprint-build/formats/`). They are the source of truth. If this skill and a format disagree, the format wins.
5. Find the sprint: the plan in `.sprint/` on `origin/main` with `status: draft`, `v{N}`. Find the build-order issue (`gh issue list --label build-order --label v{N}`) and read its `## Verify` section (`gh issue view <N> --json body`): the provision and check commands.
6. Read the tickets and their blockers:
   `gh issue list --label ticket --label v{N} --state open --limit 500 --json number,title,body,blockedBy`
   With no argument, build every open ticket. With ticket numbers, build those and every open ticket that blocks them, down the chain.
7. Find what is already built. A ticket with an open PR on a `ticket/<N>-…` branch is built: skip it, and build on its branch when it blocks another ticket. A ticket whose open blocker has no PR and is outside this run waits for a later run.
8. List the gitignored files the build needs: the files the provision and check commands read that `git check-ignore` reports. Most repositories have none.
9. Make a build order that builds as many tickets in parallel as it can. A ticket starts when its blockers are built. Two tickets can build at the same time when they change different parts of the same file; when they change the same part, one waits for the other. The Context lines and the Change section show where each ticket lands.
10. Show the build order, then build it. Do not wait for an answer.

## Build

Work through the build order. A ticket starts as soon as its own blockers are collected; it does not wait for the rest of its wave.

Run to the end without a person. The build order is your checklist: keep it current as each ticket is dispatched, collected, stopped or waiting. Stop only when nothing can move without the user. These are not reasons to stop: a summary that announces the next step, an offer to carry on, a list of choices none of which blocks you, or a sense that the turn is long or a milestone is done. Put progress updates in the same message as the next action. Wait for running agents and commands, and collect their results before you treat their work as finished. Read a format again before you write its artifact if it is no longer in your context. If one ticket is blocked, continue the independent ones. The run ends when every ticket has a PR, or is stopped or waiting with a reason.

### Dispatch

A worktree starts from the checkout's `HEAD`. A ticket's base is the branch of its blocker, or `origin/main` when it has no open blocker.

Group the tickets you start by base. For each base: `git switch --detach <base>`, check that the tree is clean apart from `.claude/worktrees/`, then send one `implementer` per ticket in that group, in one message, each with its filled prompt.

### Collect

When an implementer returns:

1. If it reports the ticket as `blocked`, stop that ticket and every ticket it blocks. The rest continues. If it reports `failed`, read why. If the cause is the attempt, not the ticket, send the ticket once more to `implementer-opus` with the retry prompt, the first report and the first worktree; keep that worktree until the retry is collected. If the retry fails too, stop the ticket and every ticket it blocks.
2. Take the directory from the result's `worktreePath`, or from the report's item (a). Check that it is a linked worktree of this repository, not the main checkout: `git -C <dir> rev-parse --path-format=absolute --git-dir` and `--git-common-dir` differ, and the common dir is this repository's `.git`. If not, the implementer wrote into the main checkout. The tree was clean at dispatch, so every path that `git status --porcelain` now lists, apart from `.claude/worktrees/`, is its work: restore those paths (`git restore` for tracked files, `git clean -fd -- <path>` for new ones, never a bare `git clean`), report it, and dispatch the ticket again once. If it happens again, stop the ticket.
3. Read the diff of every test file in the worktree. A test the ticket does not name on its `test:` line must not be weakened, skipped or deleted. If one is, do not collect the ticket: dispatch it again once and name the test. If the second attempt does the same, stop the ticket.
4. Name the worktree's branch `ticket/<N>-<slug>`. Commit the change in the worktree as one commit per `commit-format`: the subject the implementer suggests, `Closes: #N`, a `Story:` and `ADR:` line per ID on the ticket's `Related to` line, and `Assisted-by:` for the agent that built it.
5. Put the ticket on the PR of its blocker, or on `main`. A stack is one line: if a ticket has two blockers in this run, or shares its blocker with another ticket, put it on one of them, and choose from the tickets. If that is not the branch the ticket was built on, rebase the commit onto it, and run the check commands on the result.
6. If the rebase conflicts, resolve it yourself: you hold the context of both tickets. Read both tickets and both commits, and keep both intents; do not invent new behaviour. If a lockfile conflicts, resolve the manifest first, keeping both changes, then run the provision command: the package manager writes the lockfile. Never edit a lockfile by hand, never commit a conflict marker, and never abort the rebase to skip the conflict. If you cannot resolve it, stop that ticket and report it. Run the check commands, then continue the rebase.
7. Check that the branch carries exactly one commit over the branch below it: `git -C <dir> log --oneline <base>..HEAD` shows one line.
8. Push the branch. Open a draft PR per `pr-format`, with the Evidence from the implementer's report:
   `gh pr create --draft --base <branch below, or main> --head ticket/<N>-<slug> --label needs-review --title "<commit subject>" --body-file <file>`
9. Join the PR to its stack: `gh stack link --base main <PR below> <PR>` for the second layer, then `gh stack link --base main <stack number> <PR>` for each layer above it.
10. Remove the worktree.

## Stop

Show the result:

- per stack: each PR, with its ticket, its link and its state
- every ticket that stopped, with the reason
- every ticket that waits for a later run, with the blocker it waits for

Name `/sprint-review` as the next step, in a fresh session. Never review, never merge.

## Subagents

| Agent | Prompt | Use it for |
|---|---|---|
| `implementer` | `${CLAUDE_PLUGIN_ROOT}/skills/sprint-build/prompts/implementer-prompt.md` | building one ticket in its own worktree |
| `implementer-opus` | `${CLAUDE_PLUGIN_ROOT}/skills/sprint-build/prompts/retry-prompt.md` | building a ticket again, on Opus, after the implementer failed on it |

Use only the agents in this table.

Read the agent's prompt, fill its slots, and send the filled prompt as the `prompt` of the Agent call. It is the whole task: the subagent sees nothing of this conversation. Send pointers, not copies: the ticket number and the commands. The implementer reads the ticket itself. Never pass `name` on the Agent call: a named subagent can run as a teammate in the main checkout.
