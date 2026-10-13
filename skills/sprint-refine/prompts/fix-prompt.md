# fix-prompt

The session fills every `{{slot}}` and changes nothing else. The session sends the text inside the fence, filled, as the `prompt` of the Agent call to `fix-agent`; for a second attempt, to `fix-agent-opus` with the two optional slots filled.

```markdown
You fix the review findings of one ticket PR, for a session that fixes several PRs in parallel. The ticket is built; a review found what must change. The session amends the ticket's commit with your work.

<ticket>{{#N}}</ticket>

<pr>{{#P}}</pr>

<findings>{{every Fix finding of the PR's latest ### Code Review comment, verbatim}}</findings>

<provision>{{the provision command from the build-order's ## Verify}}</provision>

<check>{{the check commands from the build-order's ## Verify}}</check>

<config>{{absolute paths of gitignored files the build needs, one per line, or "none"}}</config>

[<first_attempt>{{the first fix agent's report, verbatim}}</first_attempt>]

[<first_worktree>{{the absolute path of the first attempt's worktree}}</first_worktree>]

<task>
Your worktree holds the PR head. Read the ticket with `gh issue view {{N}}` and each finding. `${CLAUDE_PLUGIN_ROOT}/skills/sprint-review/formats/finding-format.md` says what each line of a finding means. If <first_attempt> is present, read it and `git -C <first_worktree> diff` first, keep what is sound, and never edit a file there.

Work only inside your current working directory. Copy each file in <config> to the same relative path here, then run <provision> first. Never commit, and never run a git command that changes the repository.

Open every file a finding names, the code it calls and the tests around it. Make independent reads in parallel.

Fix every finding. Change only what a finding asks for; the rest of the ticket stays as it was built, and nothing new is added beyond the regression tests below. For a finding with an `Expected:` line, add a regression test that fails on the flaw and passes after the fix, beside the ticket's test. Change an existing test only where a finding says it does not prove its criterion; never weaken, skip or delete a test to make a check pass. Write a general solution, not code that only passes the test inputs.

If a finding seems mistaken, say so in a sentence in your report, with the code that shows it, and leave that part as it is.

The task is done when every finding is fixed or reported as mistaken, and the ticket's test, the regression tests and every command in <check> pass.

If you cannot get the checks to pass, stop and report `failed`.
</task>

<report>
Lead with the outcome: your first line is the status, `done` or `failed`, and one sentence on why. Then report:
(a) the absolute path of your working directory, and the model ID you run on;
(b) per finding, `F-n`: fixed, with `file:line` and its regression test, or mistaken, with the code that shows it;
(c) the evidence: the command of each check and its passing output, trimmed to the lines that matter;
(d) every file you changed;
(e) for `failed`: what you tried, the failing output, and why you think it fails.
Cover the substance, but do not pad.
</report>
```
