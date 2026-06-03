# GitHub Issue Body Template

Use this template for every ticket created by `/sprint-tickets`.

## Template

```markdown
## Objective
[One sentence: what to build and why]

## Traceability
- Story: US-### [the want this ticket serves — required]
- Part of sprint-v{N} [parent Sprint Plan — required]
- Epic: #[epic issue number] / `epic:{name}` [required]
- Governing ADR: [short ADR title/pointer, or — if none]

## Context
- Relevant files: `src/path/to/file.ts`, `src/path/to/other.ts`
- Spec: `.spec/spec.md#anchor` [the section this change lives under]
- Expected behavior: [what should happen after]

## Complexity
[S | M | L]   ← AI resource cost (drives model selection), never a time estimate
<!-- Anchors (keep reproducible — two runs should agree): S = ~1-2 files, a handful of ACs, one straightforward pass. M = a few files or moderate logic, several ACs. L = many files / cross-module / intricate logic, deep review. -->

## Write-set
- creates: [new file paths this ticket adds, or —]
- modifies: [existing file paths this ticket changes, or —]

## Requirements
- [ ] Concrete, verifiable requirement 1
- [ ] Concrete, verifiable requirement 2
- [ ] Concrete, verifiable requirement 3

## Acceptance Criteria
- Given [state], when [action], then [result]
- Edge case: [scenario] → [expected handling]
<!-- OPTIONAL — only for conditional/stateful requirements, never mandatory:
- EARS: While [state], the system SHALL [response].
- EARS: Where [feature is included], the system SHALL [response].
- EARS: If [trigger], then the system SHALL [response]. -->

## Constraints
- Do NOT modify: [off-limits files/directories]
- Must use: [specific patterns, libraries]

## Dependencies
- Blocked by: #[issue number]
- Blocks: #[issue number]
```

## Principles

- One ticket = one independently verifiable change = roughly one PR
- Traceability is mandatory: every ticket points back to a story (US-###) and forward to its parent `sprint-v{N}` and epic; cite the governing ADR only when one constrains this change
- Verification defers to the build-order: the build-order issue's single authoritative `## Verify` command is run at each wave boundary — do NOT restate it as an acceptance-criterion line in the ticket
- Acceptance criteria are Given/When/Then by default; reach for EARS (`While`/`Where`/`If-then` … SHALL) only when a requirement is conditional or stateful — it is never required
- No business justification (that's in the Sprint Plan)
- No CLAUDE.md duplication — tickets contain only the delta specific to this task
- Acceptance criteria are testable, not subjective
