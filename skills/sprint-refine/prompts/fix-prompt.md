# fix-prompt

The dispatch brief for a **fix-agent** — the **`fix-agent` custom subagent** (`agents/fix-agent.md`); this brief is the whole task contract, stated in full (a fresh agent reads only this prompt — phases don't share a session). The session writes **one brief per distinct target file** (Refine 5.1.3 — the *file* is the dispatch unit, partitioned from the published comment's `Edits:` lists), injects **every finding that file owns** into the slots, and dispatches all workers in one parallel batch, each worktree cut off this PR's head. (The main-tree solo-heal / marker-fix at a halted collection is a separate **general-purpose** dispatch of `implementer-prompt` Mode B — not this agent.)

Formats are named for provenance only — everything the worker needs from them is injected into the slots (the finding text, the provision/Verify commands, the `fix(scope):` subject shape); the worker never needs to resolve a format file.

---

## Prompt

> ## Fix: [TARGET FILE] — [N] finding(s)
>
> Fix every finding listed below to a green self-verify. They share this file — you own all of them; apply them coherently. You write code; the session collects and commits it — you produce no diff and do **not** commit anything.
>
> ### The findings — already arbitrated, do not re-litigate
> [Per finding, in order: description · evidence · its `Expected:` oracle when present (always present when testable) · commit-pinned permalink · its `Testable:` verdict · its slice of the review standard (resolved from the finding's `Violates:` pointer when it carries one, else the session's derivation — read-only context, the bar the fix must clear; when the slice is empty, the finding's description carries the bar on its own).]
>
> These findings were scored and published — a settled work order, not a proposal. Fix exactly what each describes; do not re-judge whether any is valid. Fix **every finding listed in this brief — all of them, none optional** (a SUCCESS with a listed finding unfixed is a false report); and fix **only** these — never widen scope to neighbouring code, and never pull in an unlisted finding.
>
> ### Coding standards
> {{coding_standards}}
>
> The user's own conventions, if installed. Follow them — the fixes, and their regression tests when due, are new code. If the slot is empty, follow existing codebase patterns only.
>
> ### Testable findings — regression tests
> For each finding marked `Testable: yes`, **also write a regression test** — the finding's `Expected:` line is the oracle: write the test to encode exactly that behaviour, failing against the current code and passing once that fix lands, proving the fix. Test-after the fix is the baseline; writing the test first is a fine upgrade. A `Testable: no` finding gets a plain fix — no test expected. This file's fixes **and their regression tests are one change** — the session commits them as a single commit.
>
> ### Your working copy — the standing contract
> You are working in a fresh, isolated copy of the repository (your shell's working directory) — a linked git worktree of the team repository.
>
> - **Never run git. Never commit.** You are hands, not author. Your worktree shares the team repository's refs and object store — a git write there mutates the project's shared history, which is the session's alone. Writing the files is the whole of your job; the session collects and commits your work after you return.
> - **It ships no project dependencies.** Run the **provision** command first, every time, then **Verify** — never lean on a `node_modules` that leaked from the parent tree (fragile, and unsound the moment a fix touches a dependency). Both commands come from the build-order's `## Verify` section, verbatim:
>
>   provision:
>   {{provision_command}}
>
>   verify:
>   {{verify_command}}
>
> - **Self-verify to green.** Run Verify and make it pass before you report. The gate checks form only (typecheck/compile) — it does not execute your regression tests: run each new test directly once to watch it pass (your confidence); the test lands with the fix for the corridor to run post-land. A red Verify is not a finished job.
>
> ### What you return
> 1. **Your working directory** — the absolute path of the copy you worked in (run `pwd`). The session needs it to collect your changes.
> 2. **A summary** — what you changed per finding (each fix, plus its regression test when testable) and a suggested `fix(scope): <the file's findings>` subject (plus a body only if a change-local "why" needs one). Do **not** add trailers — the session adds them. Do **not** paste a diff.
> 3. **Your status** — exactly one:
>    - **SUCCESS** — every listed finding is fixed, Verify green, self-review clean (and each regression test passes, when testable).
>    - **NEEDS_CONTEXT** — a fix is blocked on a specific missing fact (name it and the finding it blocks).
>    - **BLOCKED** — a referenced file/symbol/dependency does not exist, or a finding cannot be fixed as described (name which, and why). A red Verify you cannot resolve is BLOCKED, not SUCCESS.
>
> Before you report, check your own work against each finding: the described issue resolved, the standard satisfied, no unrelated code touched, no new issue introduced.
>
> ### What the session does after you
> Using your reported working directory, the session commits your tree inside the worktree — **one atomic commit for this file's fixes** from your suggested subject, its own trailers — and cherry-picks it onto the PR's branch before running the verify gate.
