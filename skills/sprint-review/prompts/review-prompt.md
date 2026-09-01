# review-prompt

The common envelope dispatched at 4.3.1 — one cover note sent to the whole specialist roster in a single parallel batch. It is **thin by design.** Each reviewer agent file already owns its own mandate, its calibration, its self-assessment, and its output shape; this envelope adds only what no single agent file can know: the shape of *this run* and the names of the *other specialists* sharing it. Everything substantive arrives as injected context (named below) — never reproduced here.

---

## This run's roster

Your peers in this batch:

- **Floor (always):** `code-quality-reviewer`, `code-simplifier`
- **Mandatory on sensitive surfaces:** `security-reviewer` — non-optional whenever the change touches auth · secrets · input-handling · subprocess · network (the 4.2.3 hard floor)
- **Conditional (only those the change triggered):** `silent-failure-hunter`, `type-design-reviewer`, `test-coverage-reviewer`, `comment-analyzer`, `history-reviewer`, `standards-reviewer`

Your agent file owns your lane against each peer — work it, and trust them to work theirs.

## What the dispatch injects (context, not restated here)

Held in your prompt for this review only — read each, skip silently if absent:

- **The diff** — the target. The change you are reviewing.
- **The review standard** — the `.spec` slice for the touched modules, the governing `.adr` set in full, and the `.brief` quality goals. Judge against this.
- **The platform-as-fact line** — the framework, asserted bare (e.g. `Platform: Next.js 16 App Router`).
- **Your read-surface path** — one literal labelled line, `Read-surface: <absolute path>` (this PR's review worktree). Run every Read/Grep and any file access under that path — never your shell's default working directory, which may sit at another PR's head. If no `Read-surface:` line is present in your dispatch → stop and report it; never fall back to your default working directory.
- **Your matched rules** — standards / security rule content, injected only into the specialists that use them.

These are context, not part of this envelope; the envelope names them so you know what to expect, nothing more.

You may read the local tree at your discretion — direction from this dispatch, discretion to you (your agent file holds how).

**Cite locations by the real tree line, not the diff.** The diff is the seed; the `Read-surface:` tree (above) sits at the PR head, and every `file:line` you report is the line in *that tree* (grep the symbol) — never a `gh pr diff` hunk offset, which overshoots the file and breaks the permalink.

## Output

Report in your own agent-file format. The session collects every report per `finding-report-format` (`${CLAUDE_PLUGIN_ROOT}/skills/sprint-review/formats/finding-report-format.md`) — that format owns the collected shape; you fill your block, the arbiter fills its.

Two finder-block fields do heavy lifting downstream — fill them at the source: **`expected`** (the correct behaviour, one line — the oracle a regression test will encode) on every behavioural claim; a behavioural finding without it cannot tag testable. **`violates`** (the `ADR-###` / `.spec#anchor` / named rule you judged against) whenever a named contract grounds the finding — you hold the review standard; downstream only holds what you write.
