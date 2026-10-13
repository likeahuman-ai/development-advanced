# spec-writer-prompt

The session fills every `{{slot}}` and changes nothing else. The session sends the text inside the fence, filled, as the `prompt` of the Agent call to `spec-writer`.

```markdown
You bring the system specification up to date with what one sprint changed in the code, for a session that closes the last sprint before it plans the next one. The session commits your work.

<mode>{{update | create}}</mode>

<sprint>{{v{N}}}</sprint>

<diff>{{the command that shows the sprint's change to the code}}</diff>

<task>
`${CLAUDE_PLUGIN_ROOT}/skills/sprint-plan/formats/spec-format.md` is the shape of `.spec/spec.md`, what earns a line, and how it may change. Read it first.

Your worktree holds the code with the sprint's change in it: that code is the truth. Work only inside your current working directory. Never commit, and never run a git command that changes the repository; reading with `git diff`, `git log` and `git show` is fine.

In `update` mode: run the diff command and read the changed code, the code around it, and the sections of `.spec/spec.md` that describe it. Patch only the sections the change affects. When you find a line the sprint did not cause but the code contradicts, fix it too and report it as drift.

In `create` mode: there is no spec yet. Read the whole codebase at the level the format asks for, and write `.spec/spec.md` with all nine sections.

Set the frontmatter to today and <sprint>. Make independent reads in parallel.

The task is done when every line in a section you touched is true of the code, and every section the change affects is touched.

Deliver what was asked, at the scope intended: the spec. Change no other file.
</task>

<report>
Lead with the outcome: your first line is the status, `done` or `failed`, and one sentence on why. Then report:
(a) the absolute path of your working directory;
(b) per touched section, by its exact heading: added, changed or removed, in one line each;
(c) every drift you fixed, one line each;
(d) the sections you left as they were;
(e) a commit subject per `${CLAUDE_PLUGIN_ROOT}/skills/sprint-build/formats/commit-format.md`.
Cover the substance, but do not pad.
</report>
```
