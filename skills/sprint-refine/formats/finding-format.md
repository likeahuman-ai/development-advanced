# Review Finding Format

`/sprint-review` posts findings as a top-level PR comment via `gh pr comment`; `/sprint-refine` parses that comment. This is the format.

> Sync-guard: the literal `### Code Review` heading is the parse anchor shared by both skills — `/sprint-review` must emit it verbatim and `/sprint-refine` keys off it. Never rename or reword it.

> Below-threshold findings (priority 50–74) are NOT parsed here — `/sprint-review` dumps them to the findings backlog (`.sprint/backlog.md`). This format covers only the **≥ 75** findings published for the current cycle, all of which `/sprint-refine` must fix.

> Each finding carries an explicit **`Files:`** line listing **every** path its fix touches. The permalink points at the *primary* location only; a fix that spans two files (e.g. a config change + its manifest entry) is invisible on its second file unless `Files:` names it. `/sprint-refine` groups findings by file to route same-file fixes to one implementer (no parallel clobber) — that grouping is only sound if every touched path is declared. Emit `Files:` always; one finding may list several paths.

> Each finding also carries a **`Type:`** (`security` | `correctness` | `maintainability` | `perf` | `types`) and a one-clause **`Why:`** (the consequence). **Severity gates the merge:** `[Critical]` (priority 90–100) blocks the merge; `[Important]` (priority 75–89) must be fixed this cycle. Both are published and both must be resolved by `/sprint-refine`.

## Example Review Comment

```markdown
### Code Review

Found 3 issues:

1. **[Critical]** Unhandled promise rejection in webhook handler — found by silent-failure-hunter

   https://github.com/owner/repo/blob/abc123/src/api/webhook.ts#L45-L52
   Files: src/api/webhook.ts
   Type: correctness
   Why: a failed sync is swallowed, so the caller treats a broken webhook as success

2. **[Important]** Type assertion masks potential null — found by type-design-reviewer

   https://github.com/owner/repo/blob/abc123/src/lib/parser.ts#L23-L25
   Files: src/lib/parser.ts
   Type: types
   Why: `as` hides a real null path that throws at runtime on malformed input

3. **[Important]** Cadence setting unvalidated → busy-loop — found by code-quality-reviewer

   https://github.com/owner/repo/blob/abc123/src/config.ts#L67-L72
   Files: src/config.ts, package.json
   Type: correctness
   Why: a zero/negative cadence spins the worker pool at 100% CPU

---

Reviewed by: silent-failure-hunter, type-design-reviewer, code-quality-reviewer, code-simplifier
Findings shown: priority ≥ 75 (50–74 → findings backlog; below 50 dropped)
```

## Parsing Rules

1. Look for the most recent comment starting with `### Code Review`
2. Each finding is a numbered list item with:
   - Severity in brackets: `[Critical]` or `[Important]`
   - Description after the severity
   - Agent name after "found by"
   - GitHub permalink on the next line (extract file path and line range)
   - A `Files:` line listing every path the fix touches (comma-separated)
   - A `Type:` line (`security`/`correctness`/`maintainability`/`perf`/`types`)
   - A `Why:` line (one-clause consequence)
3. The permalink format: `https://github.com/{owner}/{repo}/blob/{sha}/{path}#L{start}-L{end}`
4. Extract the line from the permalink: line = `L{start}`. Take the **fix's file set from the `Files:` line** (authoritative for grouping). If `Files:` is absent (a comment from before this field existed), fall back to the single `{path}` from the permalink.

## Clean Review (No Findings)

```markdown
### Code Review

No issues found. Reviewed for: code quality, silent failures, type design, simplification.

Findings shown: priority ≥ 75 (50–74 → findings backlog; below 50 dropped)
```

If this format is found, report "Review was clean — nothing to fix."
