# pr-format

The shape of one PR body — the `--body-file` `/sprint-build` writes per divided PR chain at close (3.4.2). One body per PR. Markdown, dense, AI-first (*artifacts are for AI, not humans*): no sign-offs, no approval boxes, no restated requirements, no IDs nothing references.

The PR is the durable envelope Review and Refine read in a fresh session (*phases don't share a session*). It summarises what Build decided; it does not prove it — code is ground truth, read past the diff in the local tree (*trust the artifact*). Reference, never reproduce: `#N` · `US-###` · `ADR-###` are pointed at, never copied. The one paste is `## Waves` — a verbatim transcription of the build-order's `## Parallel Waves` (the structural fact has no other home in the PR), not a restatement of an artifact the reader can follow a link to.

## Title

Conventional semantic subject — `type(scope): summary`, no "and". `type` = the dominant ticket kind across the PR's group; `scope` = the touched subsystem. Not a body section; passed as `--title`.

## Sections

Fixed order — consumption order, skim-first to negative-space-last. **All eight are mandatory**; an empty one carries `None` or `N/A` so absence reads as intent, not omission.

| # | Section | Holds | Consumed by |
|---|---------|-------|-------------|
| 1 | `## Summary` | 1–3 factual sentences: what was built (feature/fix/refactor), why (problem solved, goal met) | Review claim cross-ref · Refine acceptance surface (5.2.1) |
| 2 | `## Tickets` | one `Closes #N — title (US-###) — <touched path/area>` line per closed issue — `#N` + title exactly as issued, story ref in parens, then the code locus that issue lands in (the path or subsystem a refiner edits to act on it) — confirmed-from-diff or marked `inferred:`, see the locus note | Review scope · Refine scope confirm (5.0.4) · Refine fix scope (5.1.x) — never the spec sweep: 5.3.1 reads the landed diff, not PR bodies |
| 3 | `## Waves` | direct copy of the build-order's `## Parallel Waves` (pinned, 2.5.6) — hard-dep grouping per wave | Review cross-wave integration |
| 4 | `## Approach` | one paragraph: pattern followed, key design decisions, risk mitigations; cite `ADR-###` where a decision is recorded | Review claim cross-ref · Refine scope hints |
| 5 | `## Focus Areas` | where to look — subsections below | Review specialist attention · Refine fix scope |
| 6 | `## Risk Flags` | per risk class (breaking changes, migrations, behaviour changes): `yes/no` · a verdict on each `yes` · detail — verdict vocabulary in the risk-verdict note | Review prioritisation · Refine spec/fix strategy |
| 7 | `## Skip List` | matchable do-NOT-flag classes reviewers suppress — generated, lock, formatting-only; each entry a mechanical matcher (path glob, extension, or change-type class) — see the binding-constraints note | binding — see note |
| 8 | `## CI Coverage` | what automated checks already validate + whether new code added tests — see the binding-constraints note | binding — see note |

`## Focus Areas` subsections, all present:

- `### Security` — specific attack surfaces checked
- `### Edge Cases` — boundary conditions, error paths
- `### Integration` — how new code connects to existing; entry/exit points
- `### Wave Interactions` — inter-ticket contracts, three kinds (need 4 makes them load-bearing — structural, not narrative):
  - **cross-wave** — one line per contract a later wave reads from an earlier one, `<owner #N> publishes <stable surface> → <consumer #N> reads it`: which ticket OWNS the surface, which CONSUMES it, and the exact surface held stable (a signature, key namespace, table, type). `None` if no wave consumes another's output
  - **intra-wave** — the `## Waves` section asserts same-wave tickets ran in parallel; that asserts they're independent. Make the assertion checkable: per wave with >1 ticket, state `Wave K: #A ∥ #B — independent` (no shared surface), or name the hidden coupling `Wave K: #B reads <surface> from #A` so the reviewer can judge whether the parallelism was sound (a within-wave order the waves grouping hides). `n/a` for single-ticket waves
  - **collection overlap** — one line per same-line overlap healed at a cherry-pick halt (logged at collection, 3.2.4): `#A × #B on <file> — healed at pick`. The coupling signal that feeds the next decomposition. Omit when no halt occurred

**Binding-constraints note — Skip List + CI Coverage.** These two are binding, not advice: `/sprint-review` treats anything they name as a hard do-NOT-flag (suppressing known false-positive sources), and Refine holds them do-not-touch through land. A fresh-session reviewer applies them mechanically, so each entry must be *matchable* and *traceable* — not a bare label:

- **Skip List — each entry is a mechanical matcher, never a bare label.** A reviewer suppresses by matching, so each line resolves to something the file tree or diff can match: a path glob (`Generated: src/generated/**`, `Lock files: pnpm-lock.yaml`), a file-extension class (`Lock files: *.lock`), or a change-type class the reviewer can detect (`Formatting-only: whitespace/import-order hunks, no logic change`). A bare label (`Lock files`) with no matcher is not a filter — give the matcher; omit the category only if you can't express one.
- **CI Coverage — name the check, or its purpose if the id is unknown.** Best case, a line is the actual CI job / pre-commit hook id (`ci / typecheck`) so the reviewer traces what genuinely ran. When Build knows *what* a check validates but not its workflow-step id, don't fabricate one and don't bury it in prose: write `<purpose> (id unknown)` — `token type-safety (id unknown): tsc strict over the diff` — so the reviewer can still suppress that area and knows the id is unconfirmed. State what runs as fact (no "coverage is complete").
- **CI Coverage records new-code test status.** Existing checks describe pre-existing coverage; they say nothing about whether *this PR's new code* is tested. Add a `new tests:` line — `yes — <suite>` (new code under test) or `no — <why>` (e.g. covered by an existing suite, or untested → name it so the reviewer prioritises). Without it, high-risk new code (a Focus Area) reads as having no test slot at all.

**Risk-verdict note.** Each `## Risk Flags` `yes` carries a verdict so the reviewer can prioritise and the refiner can pick a fix strategy (need 3):

- `intended` — a deliberate change Build accepts as correct; the refiner preserves it, the reviewer need not treat it as a regression.
- `scrutinize` — Build is uncertain it's safe; the reviewer judges it and the refiner prepares for a possible fix.

A verdict must reflect a Build decision, not a coin-flip — if the source material describes the risk but doesn't say which it is, the honest verdict is `scrutinize` (defer the judgment to the reviewer, never assert `intended` on a guess). Free text with no verdict loses the direction need 3 requires.

**Locus note.** A `## Tickets` locus is the path/area the diff actually touched — read it off the diff, don't infer it from the Approach prose. A confirmed locus is bare (`src/import/parser/`); a locus Build can only guess carries an `inferred:` prefix (`inferred: src/auth/session.ts`) so the refiner treats a guessed area as a hint to confirm, not a verified map. The spec-writer never reads this body — the 5.3.1 sweep works from the landed diff alone.

## Template

```markdown
## Summary
<1–3 sentences: what + why>

## Tickets
- Closes #N — <title> (US-###) — <touched path/area | inferred: <area>>

## Waves
<paste of build-order ## Parallel Waves>

## Approach
<one paragraph: pattern, decisions, mitigations; ADR-### where recorded>

## Focus Areas
### Security
<attack surfaces>
### Edge Cases
<boundaries, error paths>
### Integration
<entry/exit points>
### Wave Interactions
- cross-wave: <owner #N> publishes <stable surface> → <consumer #N> reads it
- intra-wave: Wave K: #A ∥ #B — independent  |  Wave K: #B reads <surface> from #A

## Risk Flags
- Breaking changes: <yes/no — [intended|scrutinize] — detail>
- Migrations: <yes/no — [intended|scrutinize] — detail>
- Behaviour changes: <yes/no — [intended|scrutinize] — detail>

## Skip List
- <category>: <path glob | *.ext | change-type class>

## CI Coverage
- <real CI job/check | <purpose> (id unknown)>: <what it validates>
- new tests: <yes — suite | no — why>
```

## Example

Title: `feat(import): CSV bulk-import pipeline`

```markdown
## Summary
Adds a streaming CSV bulk-import pipeline for the contacts subsystem: parse,
validate, and batch-upsert rows behind a job queue so large files import
without blocking the request. Closes the gap where imports over ~2k rows timed out.

## Tickets
- Closes #203 — Streaming CSV parser with row-level validation (US-014) — src/import/parser/
- Closes #204 — Batch upsert adapter over the contacts repo (US-014) — src/import/adapter/, src/contacts/repo.ts
- Closes #205 — Import job endpoint + progress polling (US-015) — inferred: src/import/routes.ts, src/import/job.ts

## Waves
Wave 1: #203, #204 (parallel)
Wave 2: #205 (sequential)

## Approach
Adapter pattern over the existing contacts repo (ADR-009) so the importer never
touches persistence directly; parsing streams row-by-row to cap memory, and
upserts run in fixed-size batches (ADR-011). Malformed rows are collected, not
fatal — the job completes partial and reports per-row failures.

## Focus Areas
### Security
CSV injection on cell values reaching downstream sinks; upload size + MIME bound
at the endpoint; no formula-prefix passthrough.
### Edge Cases
Empty file, header-only file, duplicate keys within one batch, a row that fails
validation mid-stream, batch partial-failure rollback boundary.
### Integration
Entry: `POST /imports` (#205). Exit: contacts repo via the batch adapter (#204).
Job state polled at `GET /imports/:id`.
### Wave Interactions
- cross-wave: #203 publishes the validated row struct `ParsedRow` → #205 reads it off the parse stream
- cross-wave: #204 publishes the `upsertBatch(rows)` adapter signature → #205 calls it from the job loop
- intra-wave: Wave 1: #203 ∥ #204 — independent (#204 codes against the published signature, not #203's output)

## Risk Flags
- Breaking changes: no
- Migrations: yes — intended — adds `import_jobs` table; forward-only, see ADR-011
- Behaviour changes: yes — scrutinize — `POST /imports` now returns 202 + job id instead of 200 + body; confirm no caller blocks on the old synchronous response

## Skip List
- Generated: src/generated/**
- Lock files: *.lock, pnpm-lock.yaml
- Formatting-only: src/import/fixtures/*.csv (regenerated, no logic)

## CI Coverage
- ci / typecheck: tsc strict over the diff
- ci / lint: eslint + prettier check
- pre-commit / test: parser + adapter unit suites, import endpoint integration test
- new tests: yes — parser row-validation suite + adapter batch-boundary suite; the partial-failure path (Edge Cases) is covered by the endpoint integration test
```

## Deviates when

- it restates a fact owned elsewhere — copies a DoD instead of leaving the per-PR DoD to `.brief` (5.2.1), restates a ticket's body instead of pointing at `#N`, or redefines the waves instead of pasting the build-order's
- it adds human ceremony — sign-off box, approval checklist, reviewer-instructions preamble, restated requirements, IDs nothing references
- it prescribes process — how to dispatch, when to lift draft, how to land; the producing skill owns that. The draft state and `needs-review` label are forge facts the skill sets, not body shape
- a mandatory section is dropped rather than filled with `None`/`N/A`
