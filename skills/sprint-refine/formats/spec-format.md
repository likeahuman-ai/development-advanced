# Spec format

The shape of `.spec/spec.md` — the **System Specification**, the durable claim about how the system **is built in landed code**. One file, the conceptual layer paired with the physical one (*living knowledge infrastructure*). It is the one artifact *trust the artifact* does **not** extend to wholesale: the `.spec` is a claim *about* code, so code is ground truth and consumers verify on contact (Tickets 2.1.1, Review 4.1.4) — drift detection, not distrust. Written for a model to read its way into the current system shape (Plan, Tickets, Build, Review all read slices), never a human design doc — no sign-off, no restated decisions, no aspirational future state.

## File

- **Location:** `.spec/spec.md` — one file. (Dotdir `.spec/`; file `.spec/spec.md`.) Sharding into `.spec/<section>.md` is **cap-triggered only** (a section outgrows the file), never anticipatory; the level-2 anchors stay the addressing contract whether one file or many.
- **Frontmatter:** `last_updated: YYYY-MM-DD` and `sprint: v{N}` — the patch metadata, set whenever the spec is touched. They record *when* and *under which sprint* the spec last matched code, so a consumer can spot a spec lagging the trunk. The spec is patched in **one sweep at sprint close** (Refine 5.3.1; a sprint that parks a PR sweeps its landed subset and completes in the heal run's sweep), so between a sprint's first land and its spec sweep the trunk carries landed code the spec doesn't yet describe — a bounded, expected lag this frontmatter is built to signal.
- **Always describes what IS.** Every line is true of the **landed** code at the last patch — never planned, never "will". A section with no built reality reads "None yet" — never a speculative stub of planned content (the anchor contract below keeps all nine sections present).
- **No changelog, no archive.** `git log -p .spec/spec.md` is the history; the trailers on each `docs(spec)` commit (`Ticket: #N`, `ADR: ADR-###`) trace why each fact changed. The file holds only current state — no `@`-dated entries, no "previously" notes.

## Anchor contract — the load-bearing rule

The **nine level-2 headings are durable pointers.** Tickets, plans, and other specs address the spec by slug (`.spec/spec.md#api-surface`, `#stack`); those inbound refs rely on the slug staying put. A heading's slug is therefore a **contract** — its text never changes without updating every inbound reference. `###` subsections are **load-bearing addressing only** under `## Crosscutting Concepts & Patterns` (each concept is a durable `###` slug that parts point at). The other eight sections carry **no addressable `###`** — their density form is a flat list (entity lines, endpoint lines, path lines), so an entry is addressed by its list line, never a sub-anchor. A long `## API Surface` may group endpoints under a **bold inline label** (`**Webhook handlers**` then its endpoint lines), never a `### Webhook Handlers` heading: a `###` there would mint a sub-anchor the contract forbids, and the recoverable shape of an API Surface entry stays the flat `method + path + purpose` line whether grouped or not. All nine `##` sections are present in any complete spec (greenfield creation writes all nine; a section may read "None yet" rather than vanish, so a reader knows the shape is complete and that section is genuinely empty).

The nine, in order:

| `##` Anchor | Slug | Holds | Density form |
|---|---|---|---|
| `## Architecture` | `#architecture` | the component decomposition — the parts and how they relate | named components + one-line role each; a relation list, not prose |
| `## Runtime / Data-flow view` | `#runtime--data-flow-view` | how a request/event moves through the parts at runtime | ordered flow steps (`A → B → C`), the live paths only |
| `## Data Model` | `#data-model` | the persistent shapes and their relations | **type notation** (entity → fields → types); relations as a list |
| `## API Surface` | `#api-surface` | the externally callable contract | **endpoint / signature list** — method + path + one-line purpose |
| `## Crosscutting Concepts & Patterns` | `#crosscutting-concepts--patterns` | system-wide patterns each part must honour (auth, errors, idempotency, logging) | `###` per concept; pattern stated once, parts that follow it listed |
| `## Stack` | `#stack` | the languages, runtimes, frameworks, services actually wired in | a list — name + role; versions only where they constrain |
| `## Directory pointer-map` | `#directory-pointer-map` | where each part lives on disk | `path/ → role` lines; the map from concept to code |
| `## Infrastructure` | `#infrastructure` | the deployed/runtime substrate (queues, datastores, schedulers) as built | a list — component + role |
| `## Constraints` | `#constraints` | standing limits the code is built around (sequencing, ordering, hard bounds) | terse declarative lines, one constraint each |

## Reference, never reproduce — and what does not belong

The spec describes **shape**, not the facts other artifacts own. Point, never copy:

- A decision and its trade-off → `ADR-###` (a `## Stack` line citing `ADR-007` for *why* Postgres, never the rejected-alternatives reasoning).
- A user-facing want → `US-###`. A ticket → `#N`. A code location → `file:line`.

Out of scope entirely: implementation detail below the pattern level · test files and test strategy · CI/CD · environment values and secrets · changelogs · testable requirements (`SHALL`/`MUST` requirement IDs — the spec is not a requirements doc) · steering/aspirational text. If it changes per sprint or per ticket, it isn't spec.

## Delta-patch shape — how a patch reads

Most patches touch only the sections the sprint's landed diff changed (one sweep per sprint, at sprint close — Refine 5.3.1). **The producer's deliverable is two things: the hunk-list and the suggested commit message** — both naming `.spec/spec.md` as the target. **The producer applies the hunks itself** — in update mode it edits `.spec/spec.md` in place with Edit, in its own worktree; the session commits that worktree's tree and lands it (creation mode is the exception: the producer returns the full file and the session writes it to disk). The applied file and the landed commit are *outcomes the session lands*, never a hand-back the session re-applies. (When the patch is shown applied — the Example below — that rendering is the apply step's result, not part of what the producer emits.)

A patch is a set of **hunks**, each machine-readably tagged and scoped to one `##` (or one `###` under Crosscutting) section. The header names the file so a reader knows which document the slugs live in; the trailing `untouched:` line names the sections the patch left byte-for-byte alone, so a reader sees exactly what changed *and* what stayed without re-reading the file:

```
target: .spec/spec.md
ADDED → ## <section>: <what now exists in landed code>
MODIFIED → ## <section>: <what the section now says vs before>
REMOVED → ## <section>: <what was deleted from the code, so deleted here>
ADDED → ### <concept> (under ## Crosscutting Concepts & Patterns): <the new concept>
MODIFIED → ### <concept> (under ## Crosscutting Concepts & Patterns): <what the concept now says vs before>
REMOVED → ### <concept> (under ## Crosscutting Concepts & Patterns): <the concept dropped from the code>
untouched: <the remaining ## sections of the nine, unchanged this patch>
```

Rules of the delta:
- **Tag by exact anchor — no abbreviation, anywhere.** A hunk's `## <section>` is the section's exact heading text (the slug it resolves to) — never an abbreviation. This is **absolute**: it overrides every readability or brevity preference, in the hunk-list, the `sections:` line, and any prose. `## Crosscutting Concepts & Patterns` is written in full — never `## Concepts`, never `## Crosscutting`. One durable anchor, one spelling everywhere (the `## Architecture`…`## Constraints` slugs are the addressing contract — `Anchor contract` above). The free-prose part of a message never names a section by an abbreviated form; if brevity and the exact anchor collide, the exact anchor wins and the prose is reworded to avoid the name, not to shorten the anchor.
- **Tag at the addressed grain — `###` changes tag by their `###`.** A change inside `## Crosscutting Concepts & Patterns` tags by the `###` concept it touches (`→ ### Idempotency (under ## …)`), not by the parent `##`, whether it **adds** the whole concept, **modifies** a line inside an existing concept, or **removes** it — the tag granularity is always the addressed concept, so a cross-sprint hunk-list comparison reads a stable grain. The parent `## Crosscutting Concepts & Patterns` is tagged directly **only** when the `###`-list structure itself changes outside any one concept (a concept reordered, the section born). Same logic for the flat-list eight: the `##` tag covers changes to that section's lines; there is no finer addressable grain to tag (no `###` exists there).
- **Touch only sections the sprint's landed diff changed.** Unaffected sections are left alone — their anchors stay stable so downstream pointers keep resolving. The `untouched:` line accounts for every one of the nine not tagged, so the full part-list and the untouched-scope are both assertable from the hunk-list alone.
- **The hunk granularity equals the applied granularity.** A hunk describes exactly the lines it adds/changes/removes — if a hunk writes the first entry into an empty ("None yet") section, it says so (`ADDED → ## API Surface (first entry into the empty section): POST /api/webhooks …`), and every `file:line` the applied region introduces is named in the hunk that introduces it. A reader maps each applied line back to one hunk with no ambiguity.
- **Add, don't re-describe.** New reality appends to its section; an unchanged neighbour is not rewritten to "freshen" it.
- **Remove when removed.** Code deleted in the sprint's landed diff is deleted from the spec the same patch — the spec never describes gone code; a section emptied this way reads "None yet", never vanishes.
- **Drift is recorded, not silent.** When a patch catches reality the diff didn't introduce (a section the spec already got wrong — a new service, a renamed path), the patch fixes it in the same hunk-list and records the catch on the commit's `drift:` line (commit-message rules below) — never a silent correction.

Commit-message rules (the producer returns the message; the session finishes it). The message carries two parts: a **`sections:` body line** that is the machine record of what changed, and the **subject** as the one-line prose gloss. The `sections:` line is the durable delta record git history preserves (fact-trace) — the subject is its summary, never an independent claim.

- **`sections:` is generated FROM the hunk-list, kind-tagged.** One `sections:` body line lists every hunk's kind + exact anchor, derived directly from the hunk-list — not composed freely. Form: `sections: <KIND> ## <exact anchor>, <KIND> ## <exact anchor>, …` (e.g. `sections: ADDED ## API Surface, MODIFIED ## Runtime / Data-flow view, ADDED ### Idempotency`). Each entry is one hunk; the change-kind (ADDED/MODIFIED/REMOVED) and exact anchor survive verbatim into git history, so a reader of the commit alone recovers which sections were ADDED vs MODIFIED vs REMOVED — not an undifferentiated "X, Y, Z updated" set. Because the line is *projected* from the hunk-list, no token can appear that has no backing hunk, and no hunk can be omitted — the projection is mechanical, not aspirational.
- **Subject is a gloss of the `sections:` line, never a new claim.** The subject summarises only what `sections:` already lists — every section/topic token in the subject must trace to a `sections:` entry. A subject that names `schema` with no `## Data Model` hunk (so no `## Data Model` in `sections:`) is a false current-state signal and is not written. If the subject can't fit the changes without inventing a token, it names fewer sections, never an untouched one.
- **`sections:` carries the exact anchor; the subject may stay prose.** The exact-anchor rule (above) binds the `sections:` line — it is the durable kind-tagged record, so it holds the contract spelling. The subject is free prose: it may describe a change *by topic* without naming the anchor at all, but it may never spell an **abbreviated** anchor. The split is what lets the subject read naturally while the exact, machine-recoverable record lives once, on the `sections:` line.
- **Drift line, when any.** Each drift the patch caught gets one `drift:` line in the body (e.g. `drift: detected Stripe wired in outside this diff; added to ## Stack`); a patch with no drift omits the line. This is the defined slot for drift fact a consumer must recover.

A **full rewrite** (all nine sections, free-hand prose, ~200 lines) is the creation shape only — first sprint, no spec yet. After that the file is patched, never rewritten.

## Example — a delta patch

The sprint's landed diff adds a Stripe webhook endpoint to the delivery path. The producer's hunk-list:

```
target: .spec/spec.md
MODIFIED → ## Runtime / Data-flow view: terminal-failed events now route to the dead-letter store instead of dropping.
ADDED → ## API Surface (first entry into the empty section): POST /api/webhooks at src/app/api/webhooks/route.ts.
ADDED → ### Idempotency (under ## Crosscutting Concepts & Patterns): key check on the delivery mutation (ADR-004).
MODIFIED → ## Directory pointer-map: src/deliver/ now also owns dead-lettering.
untouched: ## Architecture, ## Data Model, ## Stack, ## Infrastructure, ## Constraints
```

The producer's suggested commit message:

```
docs(spec): record Stripe webhook endpoint and delivery idempotency

sections: ADDED ## API Surface, MODIFIED ## Runtime / Data-flow view, ADDED ### Idempotency, MODIFIED ## Directory pointer-map
Ticket: #142
ADR: ADR-004
drift: detected dead-letter store wired in outside this diff; corrected ## Runtime / Data-flow view
```

(The `sections:` line is projected straight from the hunk-list — four hunks, four entries, each carrying its kind (ADDED/MODIFIED) and exact anchor, so the commit alone tells a reader which sections were ADDED vs MODIFIED and none can name an untouched section. The subject glosses those entries by topic — no `## Data Model` token, since no `## Data Model` hunk exists; `## Crosscutting Concepts & Patterns` is recorded by its `### Idempotency` grain, never abbreviated to `## Concepts`. The `Assisted-by:` trailer is session-authored from the dispatch record (commit-format); identity across the rebased land rides the subject + trailers, so the fact stays traceable through the land.)

After the producer applies these hunks in its worktree (committed and landed by the session), the touched regions of `.spec/spec.md` read (the apply result, not part of the emitted hunk-list):

```markdown
---
last_updated: 2026-06-13
sprint: v7
---

## Runtime / Data-flow view
- Inbound event → validate → enqueue (`src/enqueue/`).
- Worker dequeues → attempt delivery → on 2xx mark delivered.
- On retryable failure → backoff → re-enqueue.
- On terminal failure → write DeadLetter, emit trace.

## API Surface
- POST `/api/webhooks` → verify signature, enqueue inbound event (`src/app/api/webhooks/route.ts`).

## Crosscutting Concepts & Patterns

### Idempotency
Every delivery mutation takes a required client-generated key; the server replays the stored result on a repeated key (ADR-004). Honoured by: `src/deliver/`, the backfill script.

## Directory pointer-map
- `src/enqueue/` → inbound validation + enqueue
- `src/deliver/` → delivery attempts, backoff, dead-lettering
```

The five `untouched:` sections are not rendered here — the hunk-list already asserts they are unchanged and the file still holds all nine, so a reader confirms the complete shape without seeing them.
