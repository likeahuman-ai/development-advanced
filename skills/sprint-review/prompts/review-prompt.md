# review-prompt

The session fills every `{{slot}}` and changes nothing else. The session sends the text inside the fence, filled, as the `prompt` of the Agent call to `code-quality-reviewer`, `standards-reviewer` or `security-reviewer`. The agent's own file holds what it looks for.

```markdown
You review one ticket PR for a session that reviews the sprint. The session decides what to fix.

<ticket>{{#N}}</ticket>

<dir>{{the absolute path of the worktree where the PR head is checked out}}</dir>

<diff>{{the diff command}}</diff>

<standards>{{the files that document how code is written here, one path per line}}</standards>

<task>
Read the ticket with `gh issue view {{N}}` for what the change is meant to do. Read every file in <standards> in full. Run the diff command. Open every changed file in full inside <dir>, and the code around the changed lines: the whole function, its callers, its types, its tests. Make independent reads in parallel. Every `file:line` you report resolves inside <dir>.

Judge only the lines this change adds or alters. The evidence can come from anywhere in the tree.

The task is done when every changed hunk is checked against what your role looks for.

Deliver what was asked, at the scope intended: findings. Never edit a file.
</task>

<report>
Lead with the outcome: your first sentence gives the number of findings. Then, per finding: a short title; one claim, what is wrong, where and why; `file:line`; the evidence, as a quote of the code or the rule; the correct behaviour in one line when the finding is about behaviour; the paths a fix would edit; whether it breaches a documented standard (cite the file and the rule) or is a judgement call; how sure you are. Report every finding, including the ones you are unsure of: the session filters. If you find nothing, say so in one line. Cover the substance, but do not pad.
</report>
```
