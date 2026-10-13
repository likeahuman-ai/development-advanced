# Preamble

This file is the format of one commit message. A ticket lands as one atomic commit. The records of a sprint plan land as one `docs(plan)` commit, and the spec sweep of the sprint before it as one `docs(spec)` commit in the same plan PR. One reason, one commit. The session writes every message; agents only suggest a subject.

## File

- **Location:** The commit message, in git. One message per commit. The graph owns the set: boundary, count, order and what landed are facts of the log, never of a message.
- **Shape:** A subject line, `type(scope): summary`. A blank line. An optional body. A blank line. Trailers, `Key: value`, one per line, last. A parser uses `git interpret-trailers` to read them.
- **Numbering:** None. The SHA is git's. A trailer never carries a SHA.
- **Green:** The tree builds and the build-order's `check` commands pass on every commit alone.

## Template

```
type(scope): <summary>

[<body: the why that has no other home, a few lines>]

[Closes: #N]
[Story: US-###]
[ADR: ADR-###]
Assisted-by: <role> <model> <effort>
```

One entry per slot:

| Slot | Rule |
|---|---|
| **type** | Mandatory. The closed value set is `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, or `style`. A ticket lands as `feat`, `fix` or `refactor`. Records land as `docs`. |
| **scope** | Mandatory. The package or module the change touches: `feat(auth)`, `fix(billing)`. Records take their artifact: `docs(plan)`, `docs(spec)`. |
| **summary** | Mandatory. Imperative, lower case, no trailing full stop, no "and". What is different after this commit, in one line. The plan commit's summary is `sprint-v{N}`. |
| **body** | Optional. The why that has no other home: a constant chosen, a workaround and its reason. Never what a ticket, a story or a decision already says. |
| **Closes** | Mandatory on a ticket commit, absent on a records commit. `#N`, the ticket this commit lands. A git trailer and a GitHub closing keyword at once: when the commit reaches `main`, GitHub closes the ticket. |
| **Story** | One line per `US-###`. On a ticket commit, each story on the ticket's `Related to` line. On a `docs(plan)` commit, each story the commit adds or changes. Absent otherwise. |
| **ADR** | One line per `ADR-###`. On a ticket commit, each decision on the ticket's `Related to` line. On a `docs(plan)` commit, each decision the commit adds or changes. Absent otherwise. |
| **Assisted-by** | Mandatory. One line per agent that wrote part of the change: the `name:` of its agent file, the model ID it ran on as the agent reports it, and its effort. The role is `session` when the session wrote it. A fix after review adds the fixer's line to the amended commit. Never `Co-Authored-By`: a model holds no copyright and signs no licence. |

## Mutability

- **On a branch, history is rewritable.** A fix after review amends the ticket's commit, so the ticket stays one commit through every rewrite. A squash never joins two tickets.
- **On `main`, a commit is frozen.** A wrong commit is followed by a `fix` or a `revert`, never rewritten.

## Ownership

The commit graph is the source of truth for what landed and in what order. Artifacts reference a commit by its ticket, `#N`, never by SHA: a rebase gives a commit a new SHA, and the subject and trailers carry its identity. Nobody quotes a message. A ticket closes when its commit reaches `main`.

## Example

A ticket commit, fixed once after review:

```
feat(billing): mark an order paid on checkout.session.completed

The reuse window on a replayed event is 10s. Client clocks skew by
less, and ADR-009 only sets the upper bound, so no record owns the
constant.

Closes: #84
Story: US-014
ADR: ADR-009
Assisted-by: implementer claude-sonnet-5-5 high
Assisted-by: fix-agent claude-sonnet-5-5 high
```

A plan commit:

```
docs(plan): sprint-v7

Story: US-014
Story: US-015
ADR: ADR-009
Assisted-by: session claude-opus-5-5 medium
```
