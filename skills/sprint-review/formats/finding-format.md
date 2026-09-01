# finding-format

The shape of the **`### Code Review` PR comment** — the published review output. Review 4.5.2 formats it (one per PR, posted at 4.6.3); Refine reads it as this PR's work order (5.0.4 eligibility, 5.1.1 fix-set). The literal `### Code Review` heading is the parse anchor both skills key off (its exact-text rule is below, in *Field rules*).

Contents = the **published set** only: findings scoring `≥75`, plus `50–74` testable, plus any `50–74` non-testable finding the 4.4.2 carve-outs promoted (spec-contradiction · small-and-structural). The rest of the `50–74` non-testable backlog lives in this PR's own `.sprint/findings-<g>.md` (4.5.1), never here — the comment is silent about bucketing. The arbitration (4.4.1) already happened: the binding score and `testable` verdict are reported as-given and **never mutated** here.

A consumer parses this comment for, per finding: its stable `F-n:` identity · binding score · `testable` verdict · fix `Cost:` · the `Expected:` oracle (every testable finding) · description + evidence · commit-pinned permalink · `Edits:` paths · `Violates:`/`Ticket:` traceability (when present) · finder. **Every recovered fact is a labeled field** — the block title's `F-n:` prefix (identity's one home — never a separate line), then `Score:`, `Testable:`, `Cost:`, `Found by:`, `Expected:`, `Edits:`, `Refs:`, `Violates:`, `Ticket:` — so a context-less reader recovers each AS what it is, never inferring meaning from a bare number, word, or position. Format dense for that single parse pass — no severity labels (gone by design), no sign-off ceremony, no restated diff/standard/context.

## Template

````markdown
### Code Review

<one-line scope: what was reviewed — the PR's change in a phrase>
Score 0–100, arbitrated priority. Published: ≥75, or ≥50 when Testable. <N> published.

**F-<n>: <terse finding title>**
Score: <0–100 integer> · Testable: <yes|no> · Cost: <S|M|L> · Found by: <finder>
<description — one claim, what + where + why in code>
Evidence: <backing facts; may cite code outside the diff — callers, types, tests, the whole function>
Expected: <the correct behaviour, one line — present on every Testable: yes finding; the regression test's oracle>
[permalink](https://github.com/<owner>/<repo>/blob/<40-char-sha>/<path>#L<start>-L<end>)
Edits: `<edited-path>`, `<edited-path>`, …
Refs: `<cited-path>` — <why cited; only when a non-edited code path is named in the evidence>
Violates: <ADR-### | .spec/spec.md#anchor | named rule> — <only when the finder judged against a named contract>
Ticket: <#N — only when the PR chain's trailers resolve the finding's edited paths to one realizing ticket>

**F-<n+1>: <next finding title>**
Score: <0–100 integer> · Testable: <yes|no> · Cost: <S|M|L> · Found by: <finder>
…
````

The `Score: <n> · Testable: <yes|no> · Cost: <S|M|L> · Found by: <finder>` line is the **labeled metadata strip** — one tagged line per finding, every value labeled (`Score:` · `Testable:` · `Cost:` · `Found by:`), so each recovers AS its own kind from the label, never from position. The label semantics live once in *Field rules* below.

**Zero published findings** — the comment is still posted, so a context-less reader distinguishes "reviewed clean" (marker present, the explicit zero-line below) from "not reviewed" (no marker at all). No metadata strip, no findings — the literal `0 published.` count plus the zero-line is the whole signal:

````markdown
### Code Review

<one-line scope: what was reviewed>
Score 0–100, arbitrated priority. Published: ≥75, or ≥50 when Testable. 0 published.

No published findings. <one line on what the specialists covered.>
````

## Field rules

- **`### Code Review`** — verbatim, exact case, the literal parse anchor (5.0.4 discovery, 5.1.1). Never a variant.
- **scope line** — one line naming the reviewed change; orients the reader, restates nothing the PR body owns.
- **legend line** — directly under the scope line, one verbatim line making the score self-describing: the scale (`0–100`, arbitrated priority) **and** the publish thresholds (`≥75, or ≥50 when Testable, or a 4.4.2 carve-out promotion`), then the published `<N>` count. Carried once in the header so a context-less reader reconstructs *why* each finding cleared the bar (and, with the count, whether any were published at all) — never repeated per finding.
- **`F-n:`** — every finding's block title opens with its number: `**F-n: <title>**` — the finding's stable identity, **assigned at publication (4.5.2) by the session**, never by a finder and never by the arbiter (finders report claims, arbitration adds `score`/`testable`/`cost`; identity exists only once a finding publishes). The sequence is **per PR and append-only across review rounds**: the PR's first `### Code Review` comment numbers its findings `F-1:`, `F-2:`, … in the order the comment lists them (the ordering rule below); a later round's comment **continues from the highest `F-n` any earlier `### Code Review` comment on this PR published** — renumbering never happens and a number is never reused, so a coordinate resolves to exactly one finding for the PR's lifetime. Only published findings consume numbers — backlogged and dropped findings never held one. **Identity is per publication:** a later round that re-finds a persisting defect publishes it as a **new block consuming the next number** — the earlier coordinate keeps resolving to the earlier block; lineage across rounds rides prose (the new block's evidence may cite the old coordinate), never the number. **Citation grammar:** inside the comment a finding is `F-n`; outside it a consumer cites the compound coordinate `#<PR> F-n` (PR number + finding — e.g. `#210 F-3`); bare `F-n` is local to its own comment's PR, meaningless without it.
- **`Score:`** — the binding `0–100` integer from 4.4.1, after an explicit `Score:` label. Reported as-given; never recomputed, never mutated. The sole priority signal — a bare integer; the header legend line supplies the scale and thresholds it reads against, so the field reads AS a score, never as a confidence percentage or a severity.
- **`Testable:`** — an explicit `yes` or `no` per finding, after the `Testable:` label — a **verdict on the finding** (a behavioural claim expressible as a test, 4.4.1), present on **every** published finding regardless of score. Never inferred from an omitted field, never read off the `Edits:` list (a test file there is unrelated metadata). A factual verdict, never a confidence adjective ("likely"/"probably"). **Every `Testable: yes` finding carries an `Expected:` line** — the tag exists only because the finder stated the oracle (the testable⇔expected lock, 4.4.1); a `yes` with no `Expected:` is a format violation, not a style choice.
- **`Cost:`** — `S`, `M`, or `L` after the `Cost:` label — the arbiter's **fix resource-cost** (effort + blast radius, 4.4.1): same semantics as a ticket's Complexity marker, **never time and never severity**. Reported as-given. Consumed at Refine 5.1.3 as the deterministic input to each fix worker's dispatch tier (the tier rule lives there).
- **`Expected:`** — the correct behaviour in one line, after the `Expected:` label — **finder-authored (4.3.1), carried verbatim**; the oracle the fix's regression test encodes (fails on the flaw, passes once fixed). **Present on every `Testable: yes` finding**; may appear on a `no` when the finder stated one. Notation follows the ticket AC rule: Given/When/Then unless the success condition depends on prior state or an accumulated count — then EARS. Never composed at publication — the session formats, it does not invent oracles.
- **No severity labels** — never `[Critical]`/`[Important]`/`[Minor]` or any variant. Priority lives in the `Score:` integer alone, by design (`Cost:` is resource, not severity — it never reads as priority).
- **`Found by:`** — the producing specialist's role name after the `Found by:` label (a role from the 4.3.1 specialist roster — the dispatched agent's own name); attribution survives arbitration's dedup. Labeled so it reads AS attribution, never as a tag or category.
- **description** — one objective claim, what/where/why in code; no "and" (two concerns = two findings).
- **Evidence** — the facts grounding the claim; may cite code **outside the changed lines** — review reads past the diff into callers, types, tests, the whole function.
- **permalink** — a **commit-pinned** GitHub blob URL with the **full 40-char head SHA** (`headRefOid` from 4.1.4): `https://github.com/<owner>/<repo>/blob/<40-char-sha>/<path>#L<start>-L<end>`. `<owner>/<repo>` is **read from the repo's own remote** (the PR's repo, e.g. `gh repo view --json nameWithOwner` / the `origin` URL) — never inferred from the example, a directory name, or any other source. Pinned to the SHA so it survives PR head moves and land (never a branch-ref or `#files` PR-diff URL, which rot on rebase). The line anchor is **always** `#L<start>-L<end>` — a single-line finding renders as `#L<n>-L<n>` (start == end), never collapsed to `#L<n>`, so the anchor form is one shape across every producer.
- **`Edits:`** — comma-separated backtick-quoted, every path the fix **edits** — and *only* edited paths (a test the fix adds/changes is edited, so it belongs here; a doc cited only as context does not). No grouping-from-content job — a test path here says nothing about the `Testable:` verdict. **The dispatch contract Refine 5.1.3 partitions on** — not merely a grouping aid: 5.1.3 reads this list to fan out **one fix worker per distinct target file** *before* it dispatches, so `Edits:` is what decides the worker partition, not the finding count. Every published finding **must carry a resolvable target file here** — at least one edited path, sourced from the **finder block**: the finder's `where` path, plus any `cites` path the fix genuinely edits (session judgment at 4.5.2 — the arbiter adds only `score`, `testable`, and `cost`, never location). A finding with **no** resolvable target file cannot be partitioned, so it cannot be dispatched as written — that is a **format violation**, not a silent degrade. A path cited but not edited must never appear here or it corrupts the partition. The label is **`Edits:`, never `Files:`** — one label never means two things across the two work orders (the ticket format carries no path-list section at all: its unproduced `## Files` footprint was retired by the v6 audit sprint).
- **`Refs:`** — present **only when** the evidence names a **code path** the fix does **not** edit (a caller, a type the change is judged against — review reads past the diff into the wider system); backtick-quoted with a terse why. Kept off `Edits:` so cited paths are recoverably distinct from the edited set — a reader (and 5.1.3) never mistakes a cited file for a path to change. A **named contract** (an `ADR-###`, a `.spec` anchor, a rule) belongs on `Violates:`, not here — `Refs:` carries code, `Violates:` carries law.
- **`Violates:`** — present **only when** the finder stated the named contract the finding judges against (`violates` in the report, 4.3.1): an `ADR-###`, a `.spec/spec.md#anchor`, or a named rule from the review standard — a pointer with a terse gloss, never a restatement. **Refine 5.1.3 reads this line to inject the fix brief's standard slice**; when absent, the session re-derives the slice from the finding's prose (the stated fallback — absence is legitimate, silence about which rule was violated is not recoverable any other way).
- **`Ticket:`** — present **only when derivable without guessing**: at publication (4.5.2) the session reads the PR chain's commits touching this finding's edited paths; if their `Ticket:` trailers resolve to exactly **one** realizing ticket → `Ticket: #N`; several or none → omit the line, never guess. **Refine 5.1.4 forwards it to the fix commit's `Ticket:` trailer**; when absent, 5.1.4 falls back to its own derivation at collection time.
- **zero-findings comment** — when nothing publishes, still post the marker plus the header (scope + legend line ending `0 published.`) plus a literal `No published findings.` line and one line on coverage; no metadata strip is emitted (this is the signal that distinguishes a clean review from an unreviewed PR — *Zero published findings* above).
- **ordering** — by `Score:`, highest first.

## Example

A PR with three published findings. Two more findings existed but never reach the comment: a `58` non-testable maintainability note went to the backlog (`.sprint/findings-<g>.md`), and a `40` was dropped — the comment is silent about both; the bucketing logic lives at 4.4.2, invisible to the reader.

````markdown
### Code Review

Reviewed PR #214 — token refresh + session TTL handling in the auth module.
Score 0–100, arbitrated priority. Published: ≥75, or ≥50 when Testable. 3 published.

**F-1: Token refresh swallows the network error, returns a stale session**
Score: 86 · Testable: yes · Cost: M · Found by: code-quality-reviewer
On a failed refresh the catch returns the cached token; callers treat it as fresh, so a 401 from the refresh endpoint silently re-uses an expired token until the next reload.
Evidence: `src/auth/session.ts:88` does `} catch { return this.cached; }`; `src/api/client.ts:24` consumes the return as a live token with no freshness check.
Expected: Given a refresh request that fails, When a caller asks for the session token, Then the failure surfaces — no cached token is returned as fresh.
[permalink](https://github.com/likeahuman-ai/claude-mastery/blob/3f2a9c1d4b8e6705a1c2d3e4f5061728394a5b6c/src/auth/session.ts#L84-L90)
Edits: `src/auth/session.ts`, `src/api/client.ts`
Ticket: #187

**F-2: Missing-scope check lets a non-admin reach the token-rotate path**
Score: 78 · Testable: no · Cost: S · Found by: security-reviewer
The rotate handler authenticates the caller but never checks the `admin` scope, so any valid session can rotate another user's refresh token, violating ADR-008's privilege-boundary rule.
Evidence: `src/auth/rotate.ts:31` reads `req.session` but the guard at `:28` only asserts presence, not scope; the admin gate at `src/auth/guards.ts:14` is never invoked on this path.
[permalink](https://github.com/likeahuman-ai/claude-mastery/blob/3f2a9c1d4b8e6705a1c2d3e4f5061728394a5b6c/src/auth/rotate.ts#L31-L31)
Edits: `src/auth/rotate.ts`
Refs: `src/auth/guards.ts` — the admin gate this path bypasses (not edited)
Violates: ADR-008 — the privilege-boundary contract

**F-3: Refresh path has no test for the failure branch**
Score: 72 · Testable: yes · Cost: S · Found by: test-coverage-reviewer
The success path is covered but the catch branch (`session.ts:88`) has no test, so the stale-token regression above can land unnoticed.
Evidence: `src/auth/session.test.ts` exercises only the happy refresh; no case forces the endpoint to 401.
Expected: Given the refresh endpoint returns 401, When the suite runs, Then a test exercises the catch branch and fails on a silently returned cached token.
[permalink](https://github.com/likeahuman-ai/claude-mastery/blob/3f2a9c1d4b8e6705a1c2d3e4f5061728394a5b6c/src/auth/session.test.ts#L40-L58)
Edits: `src/auth/session.test.ts`
````

Worked points the example demonstrates: against the header legend, the `78` clears `≥75` and the `72` clears `≥50 when Testable`, while the backlogged `58` clears neither; the `72` is `Testable: yes` even though its `Edits:` lists `session.test.ts` (the path is metadata, not the verdict's source) — and both `yes` findings carry their `Expected:` oracle while the `no` finding carries none; the single-line `rotate.ts:31` finding pins `#L31-L31`; its `Refs:` keeps the `guards.ts` code path off `Edits:` while its `Violates:` carries the ADR-008 contract, so Refine 5.1.3 groups on `rotate.ts` alone and injects ADR-008 as the fix brief's standard slice; the first finding's `Ticket: #187` resolved from the PR chain's trailers on its edited paths, the other two resolved ambiguous and omit the line. The three blocks number `F-1:`–`F-3:` in listed (score) order; a re-review of this PR would continue at `F-4:`, and the backlogged `58` and dropped `40` never consumed a number — so `#214 F-2` resolves to the scope-check finding forever. The two omitted findings appear nowhere — a reader never learns a finding was dropped or backlogged from this comment.

A reviewed-clean PR (zero published findings):

````markdown
### Code Review

Reviewed PR #218 — the documentation-only README and changelog updates.
Score 0–100, arbitrated priority. Published: ≥75, or ≥50 when Testable. 0 published.

No published findings. The doc changes were checked against the spec and brief; no behavioural or quality issues at or above the publish threshold.
````
