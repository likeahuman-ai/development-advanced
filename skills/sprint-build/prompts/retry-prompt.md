# retry-prompt

The session fills every `{{slot}}` and changes nothing else. The session sends the text inside the fence, filled, as the `prompt` of the Agent call to `implementer-opus`.

```markdown
You build one ticket that a first attempt failed to build, for a session that builds several tickets in parallel. The ticket is decided: build it as written. The session commits your work and opens the pull request.

<ticket>{{#N}}</ticket>

<built_on>{{the tickets already built under your change, #M, or "none"}}</built_on>

<provision>{{the provision command from the build-order's ## Verify}}</provision>

<check>{{the check commands from the build-order's ## Verify}}</check>

<config>{{absolute paths of gitignored files the build needs, one per line, or "none"}}</config>

<first_attempt>{{the first implementer's report, verbatim}}</first_attempt>

<first_worktree>{{the absolute path of the first attempt's worktree}}</first_worktree>

<task>
Start from what the first attempt found. Read its report, then its work: `git -C {{first_worktree}} status` and `git -C {{first_worktree}} diff`. Find what stopped it. Check its conclusions, keep what is sound, and continue from the evidence. The first worktree is evidence, not your workspace: never edit a file in it.

Read the ticket with `gh issue view {{N}}`. `${CLAUDE_PLUGIN_ROOT}/skills/sprint-tickets/formats/ticket-format.md` says what each section means. Read the decisions on its `Related to` line in `.adr/ADR.md`.

Work only inside your current working directory: it is your own worktree. Copy each file in <config> to the same relative path here, then run <provision> first. Never commit, and never run a git command that changes the repository.

Before you change anything, open every file on the ticket's Context lines, every file the first attempt touched, and the code and tests around them as the change needs. Make independent reads in parallel.

If the ticket's test does not exist yet, write it from the criteria first, and record its result before your change. Build one criterion at a time. Write a general solution, not code that only passes the test inputs. Never weaken, skip or delete an existing test to make a check pass.

Make routine judgement calls yourself. If the ticket seems mistaken or a better approach exists, say so in a sentence in your report and continue with the ticket as written. The `Stays as it is:` line is a boundary. Don't add features, tests, files, docs or refactors that the ticket does not ask for. If you think one would help, say so at the end of your report.

The task is done when the ticket's test and every command in <check> pass.

If the ticket cannot be built as written, stop and report `blocked`. If you cannot get the checks to pass, stop and report `failed`.
</task>

<report>
Lead with the outcome: your first line is the status, `done`, `failed` or `blocked`, and one sentence on why. Then report:
(a) the absolute path of your working directory, and the model ID you run on;
(b) a commit subject per `${CLAUDE_PLUGIN_ROOT}/skills/sprint-build/formats/commit-format.md`;
(c) the evidence: the ticket's test before and after, and the output of <check>, trimmed to the lines that matter;
(d) every file you changed or added;
(e) what stopped the first attempt, what you kept, and what you did differently;
(f) for `blocked`: what in the ticket stops the build, quoted; for `failed`: what you tried, the failing output, and why you think it fails;
(g) every judgement call the reviewer must know about.
Cover the substance, but do not pad.
</report>
```
