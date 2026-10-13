---
name: sprint-tickets
description: "Turns the sprint's landed plan into ready tickets on GitHub, each with its blockers linked, plus a build-order issue that shows the waves and holds the build's checks. Runs from start to end without a person. For a team that builds in parallel stacked PRs; a solo, single-PR project uses the development plugin. Never re-plans: if the code contradicts the plan, it stops and names /sprint-plan."
argument-hint: "Nothing; it finds the sprint plan with status draft"
disable-model-invocation: true
---

# Sprint tickets

Turn the sprint plan into tickets that `/sprint-build` can build without a question. The plan is decided: never reopen it, never ask the user to confirm it. There is no interview and no approval round. The person who typed the command expects the issues on GitHub when it ends.

If the code shows the plan cannot be built as written, because a premise is false or two parts contradict, publish nothing and stop. That is a signal for `/sprint-plan`, not a reason to improvise.

Run to the end without a person. The ticket list is your checklist: keep it current as each ticket is designed, published or stopped. Stop only when nothing can move without the user. These are not reasons to stop: a summary that announces the next step, an offer to carry on, a list of choices none of which blocks you, or a sense that the turn is long or a milestone is done. Put progress updates in the same message as the next action. Wait for running agents and commands, and collect their results before you treat their work as finished. Read a format again before you write its artifact if it is no longer in your context. The run ends when every ticket and the build-order are on GitHub, or when a contradiction with the plan stops it.

**Initial request:** $ARGUMENTS

## Start

Start is your own work.

1. Run `gh auth status`. If the user is not authenticated, tell the user to run `gh auth login`, and stop. Run `gh --version`. If it is older than 2.94.0, tell the user to upgrade `gh`, and stop: blocker links need it.
2. Run `git fetch origin`. The plan landed on `main`: if the tree is clean, run `git switch --detach origin/main`; if not, read every file from `origin/main` with `git show origin/main:<path>`.
3. Read `${CLAUDE_PLUGIN_ROOT}/skills/sprint-tickets/formats/ticket-format.md` and `build-order-format.md` beside it. They are the source of truth. If this skill and a format disagree, the format wins.
4. Find the plan in `.sprint/` with `status: draft`. If there is none, stop and name `/sprint-plan`. Read it in full, then the stories it cites, every decision in `.adr/ADR.md`, and the sections of `.spec/spec.md` its Architecture cites.
5. Run `gh issue list --label ticket --label v{N} --state all --limit 500 --json number,title,body`. Tickets that already exist for this sprint are kept: this run continues where an earlier run stopped, and creates only what is missing.

## Design

1. **Assign the seams.** From the plan, `.spec/spec.md` and the epics, list the modules that more than one epic will touch. Give each one owner: the epic that defines its shape.
2. **Dispatch.** Send one `code-architect` per epic, all in one message, each with its filled prompt.
3. **Collect.** Combine the reports into one ticket list. Then check it:
   - Every ticket passes the gate in `ticket-format`.
   - No seam is designed two ways, and every seam change an architect needs is resolved.
   - No blocker exists only because one ticket reuses code another adds: move that code to where both consume it, and drop the blocker.
   - Every `file:line` an architect reported as unconfirmed exists: search for it.
   - Every story slice and scope row of the plan is covered by a ticket.
   - The blockers form no cycle.

   Fix what you can in the list. If an architect's design fails a check you cannot fix, send that architect again once, with the defect named. If a report says the code contradicts the plan, stop: report the contradiction, quoted, and name `/sprint-plan`.

## Publish

1. **Labels.** Create each with `gh label create <name> --force`: `ticket`, `build-order`, `needs-review`, `needs-refine`, `v{N}`, and one `epic:<name>` per epic, written as the plan writes it.
2. **Tickets.** Create the issues in dependency order, blockers first, so each `Blocked by` line can name real numbers. One call per ticket; write each body to a temporary file first:
   `gh issue create --title "<title>" --body-file <file> --label ticket,v{N},epic:<name> --blocked-by <N,N>`
   Replace each blocker's title with the number its issue got. If a create fails, stop and report which tickets exist.
3. **Build-order.** Compute the waves from the blockers. Read the provision command and the check commands off the repository's own tooling: its package manager, its scripts, its CI config. Never invent one. Create the issue per `build-order-format`, with the labels `build-order` and `v{N}`.

## Stop

Show the tickets with their numbers and links, grouped by wave, and the build-order link. Name `/sprint-build` as the next step, in a fresh session. Never build.

## Subagents

| Agent | Prompt | Use it for |
|---|---|---|
| `code-architect` | `${CLAUDE_PLUGIN_ROOT}/skills/sprint-tickets/prompts/code-architect-prompt.md` | designing one epic as tickets |

Use only the agent in this table.

Read the agent's prompt, fill its slots, and send the filled prompt as the `prompt` of the Agent call. It is the whole task: the subagent sees nothing of this conversation. Never pass `name` on the Agent call.
