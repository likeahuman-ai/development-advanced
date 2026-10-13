# spec-review-prompt

The session fills every `{{slot}}` and changes nothing else. The session sends the text inside the fence, filled, as the `prompt` of the Agent call to `spec-reviewer`.

```markdown
You review one ticket PR for a session that reviews the sprint. You find where the change and its ticket disagree. The session decides what to fix.

<ticket>{{#N}}</ticket>

<dir>{{the absolute path of the worktree where the PR head is checked out}}</dir>

<diff>{{the diff command}}</diff>

<task>
Read the ticket with `gh issue view {{N}}`. `${CLAUDE_PLUGIN_ROOT}/skills/sprint-tickets/formats/ticket-format.md` says what each section means. Inside <dir>, read every decision on its `Related to` line in `.adr/ADR.md`, every other decision whose subject the Change section names, and the `.spec/spec.md` section on its `spec:` line, if any.

Run the diff command. Open every changed file in full inside <dir>, the files on the ticket's Context lines, and the callers and tests of the changed code. Make independent reads in parallel.

Check the tests as closely as the code. A test proves its criterion only when it fails without the change and its expected values come from an independent source: a known-good literal, a worked example, the ticket. Two shapes prove nothing:
- **Implementation-coupled**: it mocks internal collaborators, tests private methods, or checks through a side channel. The tell: it breaks on a refactor that keeps the behaviour.
- **Tautological**: the assertion computes the expected value the way the code does, so it can never disagree with the code.

The task is done when every sentence of the Objective and the Change section, every acceptance criterion, every changed file and every decision on `Related to` is accounted for.

Deliver what was asked, at the scope intended: findings. Never edit a file.
</task>

<report>
Lead with the outcome: your first sentence gives the number of findings in each group. Then, per finding: a short title; the ticket line it rests on, quoted; `file:line`; the evidence; the correct behaviour in one line when the finding is about behaviour; the paths a fix would edit; how sure you are. Group them:
(a) requirements the ticket asked for that are missing or partial;
(b) behaviour in the diff that was not asked for, and anything the `Stays as it is:` line rules out;
(c) requirements that look implemented but wrong;
(d) criteria whose test does not prove them, and why;
(e) decisions the change breaks.
Report every finding, including the ones you are unsure of: the session filters. Cover the substance, but do not pad.
</report>
```
