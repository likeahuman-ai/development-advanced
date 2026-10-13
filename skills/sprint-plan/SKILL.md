---
name: sprint-plan
description: "Interviews the user about the next sprint until the project's records agree with it and the sprint plan is ready for /sprint-tickets to run without a person. Proposes new and changed stories and decisions, checks them against each other and the code, then lands the plan in one PR on main. Also closes the last sprint: brings .spec up to date with what landed. The only phase that talks with the user. For a team that builds in parallel stacked PRs; a solo, single-PR project uses the development plugin. Never writes tickets, never builds."
argument-hint: "An idea, or a record to start from (optional)"
disable-model-invocation: true
---

# Sprint plan

Interview the user about the next sprint until you reach a shared understanding. Sharpen the project's records as you go. Then write the sprint plan and land it.

This is the one phase with a person in it. `/sprint-tickets` and `/sprint-build` run after it with no person in the loop, so every decision they need is made here. Explore, present what you found, confirm with the user, then write.

**Initial request:** $ARGUMENTS

## Start

Start is your own work. Gather what you need to ask the first round, and no more.

1. Run `gh auth status`. If the user is not authenticated, tell the user to run `gh auth login`, and stop.
2. Set up the repository on first contact. Each act is a no-op when it is already done:
   - no repository: `git init`; no git identity: ask the user for it, never invent one;
   - no remote: `gh repo create <owner>/<repo> --private --source=. --remote=origin`;
   - `.claude/settings.json` lacks `"worktree": {"baseRef": "head"}`: add it. Every worktree must start from the branch the session checks out, or stacking breaks.
   - `.gitignore` lacks `.claude/worktrees/`: add it, so subagent worktrees never show as untracked files.

   These changes ride the plan PR.
3. Read the formats in `${CLAUDE_PLUGIN_ROOT}/skills/sprint-plan/formats/`. They say what each record holds, what earns a record, and how a record can change. They are the source of truth. If this skill and a format disagree, the format wins.
4. Run `git fetch origin`. Read `.brief/brief.md`, the plans in `.sprint/`, and any record the request names.
5. Settle the last sprint. Find the plan with `status: draft`.
   - Its tickets are all closed (`gh issue list --label ticket --label v{N} --state open` is empty): the sprint landed. Close it in Write step 1.
   - Some are open: the sprint is not finished. Tell the user which tickets are open and ask whether to finish it first (`/sprint-build` or `/sprint-refine`) or to abandon it. Abandon only when the user says so.
   - A branch `docs/sprint-v{N}` exists with no merged PR: a plan was accepted but never landed. Resume from it.
6. Find the starting point in the request.

## Interview

Interview the user until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled: the questions you can ask _now_ without guessing at answers you haven't heard yet. Ask the frontier in one round: number each question and give your recommended answer. If the frontier is long, ask the questions that unblock the most first, and keep the rest for the next round. Then wait for the user's answers.

Format a round like so:

```
**Q1 — <question title>**: <question body, with the record quoted where one applies, including multiple choices>

→ <your recommended answer>

---

**Q2 — <question title>**: …

→ …
```

Each round the user answers reshapes the tree. Recompute the frontier and ask the next round. A question whose answer depends on another open question in this round belongs to a later round.

Finding _facts_ is your job, never the user's. Facts live in the records, the formats, the code, the issue tracker and sources outside the repository. When a frontier question needs a fact that a few reads find, read it yourself. For a wide search, send a subagent (see Subagents), and ask the rest of the frontier while it runs: only the questions downstream of it wait. The _decisions_ are the user's: put each to them and wait.

The records run from broad to narrow: brief, stories, decisions, sprint plan. Settle the broad questions first.

Guide the user like a colleague who knows the history. Explain why a question matters when it is not obvious. Never grade the answers. If a session runs very long, the scope is usually too big: split it into sprints.

## During the session

### Show the records

Users do not read the record files. Quote a record exactly, with its ID, every time a question or a warning rests on it. Never refer to a record by a bare ID: the ID rides inside the quote. Quote only the part that matters.

Put the record inside the question, so that the user can only answer with it: "ADR-002 says: *once the buyer assigns a replacement, the previous occupant does not return to that seat.* Your idea lets her take the seat back. Which one wins?"

### Sharpen the language

When the user uses a term that conflicts with the brief's user roles or the words the stories use, call it out at once. When a term is vague or overloaded, propose a precise one: "You're saying 'account': do you mean the Customer or the User?"

### Stress-test with scenarios

When a domain relationship is under discussion, invent concrete scenarios that probe its edges, and make the user decide them.

### Cross-reference

When the user states how something works, check whether the code agrees, and surface a contradiction: "Your code cancels entire orders, but you just said partial cancellation is possible. Which is right?"

When you draft a record, new or changed, send `docs-explorer` with the draft to find every record it relates to. For each one it reports, ask whether the draft fits it, and propose any change in the form its format's Mutability section allows.

### Draft records as they are resolved

When a story or a decision is resolved, draft it there and then, in its format, in the conversation. Don't batch them up. Drafts live in the conversation until Write. If the session ends before that, nothing is written.

### Find the home of every answer

Go from broad to narrow, and stop at every level the answer touches:

1. Does it change the product's pitch, its roles, its quality bars or its Definition of Done? That is the brief. Say so and ask the user.
2. Is it a want? Test it against the gate in `stories-format`.
3. Is it a decision? Test it against the gate in `adr-format`.
4. Is it the sprint's goal, scope, or how the work splits into epics? That is the sprint plan.
5. Is it detail for the build? It goes in the plan's scope or architecture, so the architects can design from it.
6. None of these? It stays in the conversation.

Most gate conditions ask for judgement. If one fails, say so once, say what it costs later, and recommend a fix. If the user overrides, draft what the user asked for and do not repeat the warning. Two conditions have no override: every story names a role from the brief, and every `Related to` ID resolves.

### Warn by cost

An edit in place gets a note. A record that must be replaced gets a clear warning, with what else it touches. The strongest warning is for a plan that breaks a decision or covers something no story asks for: quote the record and say what the build will then do against it. The user can override every warning.

## Plan

When the user confirms a shared understanding, write the sprint plan per `sprint-format`, in the conversation.

1. **Explore once.** Send one `codebase-explorer` per epic you expect, in one message, for the facts the plan's scope and architecture rest on. Draft from the reports.
2. **Draft.** Goal, solution, stories, epics and scope. Every story slice and every scope row sits under one epic.
3. **Make it runnable without a person.** `/sprint-tickets` reads this plan alone. Check that no question is left open in it: no "or", "TBD" or "depending on", and no choice an architect would have to make that the user has not made. If one is left, ask it as a round.
4. **Show everything once.** Check first that every draft passes its gate or was overridden, and that every new or changed record serves the plan. Then show every new record in full, every changed record with the old text next to the new, and the plan. Ask the user to accept. If the user changes something, go back, then show everything again. Write nothing before the user accepts.

## Write

1. If the last sprint landed (Start step 5), close it first. Check out `origin/main` detached, check that the tree is clean, and send `spec-writer` with its filled prompt: `create` mode when `.spec/spec.md` does not exist, else `update` mode, with the diff from the last plan's commit (`git log origin/main --grep "docs(plan): sprint-v{N-1}" -1`) to `origin/main`. Keep its worktree until step 3.
2. Cut `docs/sprint-v{N}` from `origin/main`. If the checkout has uncommitted work, use a temporary worktree.
3. If step 1 ran: copy `.spec/spec.md` from the spec-writer's worktree into the branch, set the last plan's status to `built`, and commit both per `commit-format`, as `docs(spec): sprint-v{N-1}`. Remove the spec-writer's worktree.
4. Write the records and the plan. Set every older plan that is not `archived` or `abandoned` to `archived`. Commit per `commit-format`, as `docs(plan): sprint-v{N}`.
5. Push the branch and open a PR per `pr-format`. If you used a temporary worktree, remove it.
6. Merge the PR by its number: `gh pr merge <N> --rebase --delete-branch`. The user accepted the records in Plan step 4; that is the review.

## Stop

Show what you wrote, with the IDs and the PR link. Name `/sprint-tickets` as the next step, in a fresh session: the plan carries everything it needs. Never write tickets, never build.

## Subagents

| Agent | Prompt | Use it for |
|---|---|---|
| `codebase-explorer` | `${CLAUDE_PLUGIN_ROOT}/skills/sprint-plan/prompts/codebase-explorer-prompt.md` | facts in the code |
| `docs-explorer` | `${CLAUDE_PLUGIN_ROOT}/skills/sprint-plan/prompts/docs-explorer-prompt.md` | the records related to a draft or a subject |
| `web-researcher` | `${CLAUDE_PLUGIN_ROOT}/skills/sprint-plan/prompts/web-researcher-prompt.md` | facts from outside the repository |
| `spec-writer` | `${CLAUDE_PLUGIN_ROOT}/skills/sprint-plan/prompts/spec-writer-prompt.md` | bringing `.spec/spec.md` up to date with the last sprint |

Use only the agents in this table.

Read the agent's prompt, fill its slots, and send the filled prompt as the `prompt` of the Agent call. It is the whole task: the subagent sees nothing of this conversation. Never pass `name` on the Agent call: a named subagent can run as a teammate in the main checkout.

A report is facts. The decisions stay with the user.
