# Code Architect Prompt Template

Use this template when dispatching `code-architect` agents. Launch one per epic/feature from the Sprint Plan.

## Prompt

> Design the implementation for [EPIC/FEATURE NAME].
>
> ### Sprint Plan Slice
> [Paste the Architecture shape-of-change pointer for this epic/feature — the 2-3 line "shape of the change" from the Sprint Plan, NOT a full architecture block]
>
> ### Spec Slice
> [Paste only the touched-module sections of `.spec/spec.md` — the relevant Architecture / Data Model / API Surface entries plus the "Crosscutting Concepts & Patterns" section (auth, error-handling, logging, glossary/naming). Omit untouched modules. Skip if `.spec/spec.md` is absent.]
>
> ### Codebase Exploration Findings
> [Paste relevant findings from the codebase-explorer agents]
>
> ### Architectural Decisions (ADRs)
> [Paste the Y-statement of each governing ADR if `.adr/ADR.md` exists, otherwise omit this section]
> These are constraints from the planning phase. Treat the Sprint Plan slice, the Spec slice, and the ADR Y-statements as settled constraints — design within them. If an edge case conflicts with an ADR, the Spec, or the Sprint Plan, flag it rather than silently contradicting the decision.
>
> ### Your Job
> Produce, per ticket:
> - **US-### back-ref** — the user story (or stories) this ticket serves, by stable ID (e.g. `US-001`).
> - **Spec pointer** — the spec section this ticket implements against, as an anchor (`.spec/spec.md#anchor`).
> - **Governing-ADR pointer** (optional) — the ADR Y-statement that constrains this ticket, if one applies.
> - **epic** — the epic/feature this ticket belongs to.
> - **Write-set** — the exact file paths this ticket will touch, split into `creates` (new files) and `modifies` (existing files changed). This is the file-safety basis for parallel-wave grouping, so be precise and complete — a missed path can cause two parallel implementers to clobber the same file.
> - **depends-on** — artefacts this ticket needs to exist first, each marked HARD (won't compile/run without) or SOFT (works without, better with), referencing the producing ticket. Ordering only — distinct from the write-set.
> - **Acceptance criteria** — Given/When/Then by default; add OPTIONAL EARS (While/Where/If-then SHALL) only for conditional or stateful behaviour. Never make EARS mandatory.
> - **Complexity estimate** — S (single agent context, few files), M (full session, multiple files), L (multiple sessions, many systems).
>
> Do NOT produce per-ticket verification commands — verification is centralized in the build-order's single authoritative `## Verify` section.
>
> Complexity indicates AI resource cost, not human time.

## Anti-bloat

The design you produce lives in your returned findings and in the GitHub Issues themselves. Do NOT write a design-doc file. There is no per-cycle design document — the Issues are the durable record.

## Usage

Replace `[EPIC/FEATURE NAME]` and paste the Sprint Plan slice (the architecture shape-of-change pointer), the touched-module Spec slice, and exploration findings. Each agent gets one epic or feature — don't overload a single agent with the entire Sprint Plan.
