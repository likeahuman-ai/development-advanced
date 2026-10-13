---
name: sprint-review
description: "Reviews each ticket PR of the sprint on four axes: spec (does the code build what the ticket asks, and do its tests prove the criteria?), standards, quality and security. Runs the reviewers in parallel subagents, sorts their findings into what to fix and what to note, and posts one ### Code Review comment per PR. Runs from start to end without a person. For a team that builds in parallel stacked PRs; a solo, single-PR project uses the development plugin. Never fixes."
argument-hint: "PR numbers (optional; default: every PR labelled needs-review)"
disable-model-invocation: true
---

# Sprint review

Review each ticket PR against its ticket and against the repository's standards, and publish what you find. The reviewers report everything; this skill sorts. Never fix: `/sprint-refine` owns the fixes.

Run to the end without a person. The PR list is your checklist: keep it current as each PR is reviewed, posted or skipped. Stop only when nothing can move without the user. These are not reasons to stop: a summary that announces the next step, an offer to carry on, a list of choices none of which blocks you, or a sense that the turn is long or a milestone is done. Put progress updates in the same message as the next action. Wait for running agents and commands, and collect their results before you treat their work as finished. Read a format again before you write its artifact if it is no longer in your context. The run ends when every PR has its comment posted, or is skipped with a reason.

**Initial request:** $ARGUMENTS

## Start

Start is your own work.

1. Run `gh auth status`. If the user is not authenticated, tell the user to run `gh auth login`, and stop.
2. Read `${CLAUDE_PLUGIN_ROOT}/skills/sprint-review/formats/finding-format.md`. It is the source of truth for what is a Fix and what is a Note. If this skill and the format disagree, the format wins.
3. Find the PRs: the numbers in the request, or `gh pr list --label needs-review --limit 500 --json number,headRefName,baseRefName,headRefOid`.
4. **Pin each change.** Run `git fetch origin`. For each PR, add a temporary worktree at its head: `git worktree add --detach <path> origin/<headRefName>`. Capture its diff command once: `git -C <dir> diff origin/<baseRefName>...HEAD`. Check that the base resolves and the diff is not empty. A bad ref or an empty diff fails here, not inside the reviewers.
5. **Find the ticket.** Read the `Closes:` trailer of the PR's commit. If there is none, skip the spec axis for that PR and say "no ticket available".
6. **Find the standards.** The coding sections of `CLAUDE.md` and every file they point to; the rules of the coding-standards plugin, if the repository has them; `.brief/brief.md`, for its quality bars and its Definition of Done; `.spec/spec.md`.
7. **Find the next number.** Read the PR's earlier `### Code Review` comments. The first new finding takes the next `F-n` after the highest one used.

## Review

For every PR, send `spec-reviewer`, `standards-reviewer` and `code-quality-reviewer`. Add `security-reviewer` when the diff touches authentication or authorization, secrets, input handling, subprocesses or network calls. Send every reviewer of every PR in one message, each with its filled prompt.

## Report

Per PR, sort every reported finding into **Fix** or **Note** by the gate in `finding-format`. Within one axis, merge findings that name the same flaw. Never merge findings across axes: a change can pass one axis and fail another, and a merged finding lets one hide the other. Drop a finding only when the code in the worktree shows it is wrong.

Write the comment per `finding-format`, with the numbers from Start step 7, and post it: `gh pr comment <N> --body-file <file>`. Then move the label: `gh pr edit <N> --remove-label needs-review --add-label needs-refine`.

Remove the temporary worktrees.

## Stop

Show, per PR: its link, its ticket, the Fix and Note counts per axis, and the worst Fix finding of each axis, if any. Don't pick a single worst finding across axes. Name `/sprint-refine` as the next step, in a fresh session. Never fix.

## Subagents

| Agent | Prompt | Use it for |
|---|---|---|
| `spec-reviewer` | `${CLAUDE_PLUGIN_ROOT}/skills/sprint-review/prompts/spec-review-prompt.md` | where the change and its ticket disagree |
| `standards-reviewer` | `${CLAUDE_PLUGIN_ROOT}/skills/sprint-review/prompts/review-prompt.md` | where the change and the repository's standards disagree |
| `code-quality-reviewer` | `${CLAUDE_PLUGIN_ROOT}/skills/sprint-review/prompts/review-prompt.md` | bugs, error handling and comments that lie |
| `security-reviewer` | `${CLAUDE_PLUGIN_ROOT}/skills/sprint-review/prompts/review-prompt.md` | security flaws and data exposure |

Use only the agents in this table.

Read the agent's prompt, fill its slots, and send the filled prompt as the `prompt` of the Agent call. It is the whole task: the subagent sees nothing of this conversation. Send pointers, not copies: the ticket number, the worktree, the diff command and the standards files. Never pass `name` on the Agent call.
