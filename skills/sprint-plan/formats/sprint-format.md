# Sprint Plan Format

Use this format when writing a Sprint Plan in Phase 4 of `/sprint-plan`.

> [!IMPORTANT]
> A Sprint Plan is an **immutable per-cycle snapshot** — once a cycle is approved, the file is frozen. It is **NOT** a living Sprint Backlog. In-cycle status (what's done, in progress, blocked) lives in **Issues**, never here. A Sprint Plan **references** the Brief, Spec, Stories, and ADRs — it never reproduces them.

---

## Sections (in order)

### 1. Goal

The **first line** is a single pass/fail objective, written exactly in this shape:

> This cycle succeeds iff `<one objective>`.

One sentence. One condition. If you need "and" to join two objectives, the cycle is too big — split it.

### 2. Non-goals

Things this cycle deliberately will not do. Short bullets. Use this to cut scope drift before it starts.

### 3. Solution

One **non-technical** paragraph: what this cycle delivers, at the highest level, in plain language. No architecture, no file names, no jargon.

### 4. User-stories slice

The slice of user stories this cycle covers. **Reference** each by its stable ID (`US-###`) and add cycle-specific detail — **never restate the story sentence** (it already lives in `.stories/STORIES.md`).

- `US-012` — cycle detail: which part of this want the cycle delivers
- `US-018` — cycle detail: ...

### 5. Scope

A **single** in/out table. Do not add a standalone out-of-scope section — everything out-of-scope goes in the right column here.

| In scope this cycle | Out of scope this cycle |
| --- | --- |
| ... | ... |

### 6. Architecture (shape of the change)

The **shape of the change** only — 2–3 lines describing how this cycle alters the system and pointing into the Spec. This is a plan-time pointer, distinct from the spec's `ADDED/MODIFIED/REMOVED` delta that the spec-writer applies at `/sprint-refine`. Do **not** reproduce the architecture: no directory tree, no component catalogue, no data-flow diagram, no integration-point list. Those live in `.spec/spec.md`.

> Example: "Adds a `RetryQueue` between the dispatcher and the worker pool — see `.spec/spec.md#runtime-view`. No data-model change."

### 7. Success metric (target)

The measurable target for this cycle. **Name the Brief's north-star metric** and state the period target for it here (the Brief defines the metric and its guardrails; the period target lives here, in the cycle snapshot).

> Example: "North-star (median time-to-first-PR, defined in `.brief/brief.md`): cut from 9 min to ≤ 6 min this cycle."

### 8. Timebox

How long this cycle is allotted. One line.

### 9. Definition of Done

A **reference** to the Brief's canonical Definition of Done (`.brief/brief.md`). Do not restate it — the Brief owns the DoD. Add only cycle-specific exit conditions if any exist beyond the canonical DoD.

### 10. Dependencies & Risks (optional)

Include only when real. Dependencies = things outside this cycle's control. **Risks must point to labelled Issues** (`#123`) that track them — do not park unmanaged risk text in the snapshot.

| Dependency / Risk | Impact | Tracking Issue |
| --- | --- | --- |
| ... | ... | #123 |

---

## Do NOT include

- A full **Architecture block** — no directory tree, component catalogue, data-flow diagram, or integration-point list (Spec owns these; here, only a 2–3 line shape-of-change pointer).
- **File tables** — no Source / Detailed-changes / Migration / Verification / History / file-or-line tables. File-level detail belongs in tickets and the Spec.
- A **standalone Out-of-Scope section** — the single Scope table's right column covers it.
- A **multi-row Success-Metrics matrix** — one north-star target only.
- **User/System-Flow**, **Testing Strategy**, **Privacy & Security**, and **Rollback Plan** optional sections — dropped from this format.
- **In-cycle progress** of any kind — which ticket is done, in progress, or blocked lives in Issues, never here. (This is distinct from the frontmatter `status:` field, which tracks the coarse *cycle* milestone — see Formatting Rules.)
- Restated **story sentences**, **DoD text**, or **Spec content** — reference, never reproduce.
- **Open `[NEEDS CLARIFICATION: …]` markers** in an approved plan — they're allowed in a `draft` while you resolve them, but every one must be gone before the plan leaves `draft`.

---

## Formatting Rules

- **Save location:** `.sprint/sprint-v{N}.md` (sequential version number, no gaps).
- **Immutable per cycle, except `status:`** once approved, the body is frozen — edit nothing. The one mutable field is `status:` in the frontmatter, which advances through the cycle (see Status lifecycle below). The next cycle gets a new `sprint-v{N+1}.md`.
- **Frontmatter:**
  ```yaml
  ---
  version: {N}
  status: draft
  date: {YYYY-MM-DD}
  author: {author name}
  previous: sprint-v{N-1}.md
  ---
  ```
  Set `previous: null` for the first Sprint Plan (v1).
- **Status lifecycle:** `draft → built → archived` (+ `abandoned`). It is the only field that changes after approval, and exactly one skill owns each flip:
  - `draft` — set by `/sprint-plan` at creation. Covers planning **and** building-against: the file stays `draft` through `/sprint-tickets` and for the whole of `/sprint-build` until the code lands.
  - `built` — set by `/sprint-build` when the code is done and the PR is created. Means "implementation complete."
  - `archived` — set by `/sprint-plan`'s cascade when a newer draft is created.
  - `abandoned` — set by `/sprint-plan` if the user abandons a draft before it reaches `built`.

  There is **no `released`** (that belongs to the plugin's own `.prd/` lifecycle, not a project Sprint Plan) and **no `planned`/`complete`** — cycle-completion is signalled by the PR's `cycle-complete` label, not duplicated here.
- **References, not reproductions:** every section points at the Brief, Spec, Stories, or ADRs by anchor/ID rather than copying their content.
- **No implementation code:** Sprint Plans describe what and why for the cycle, not how at the code level.
- **Scope table:** always include both columns — what's in AND what's out.
