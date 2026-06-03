# Review Agent Dispatch Prompt

Template for dispatching specialist review agents. The main model fills in the bracketed sections.

---

## Review: [agent role]

### PR Context
- **PR:** #[number] — [title]
- **Repository:** [owner/repo]
- **Branch:** [head] → [base]
- **Description:** [PR summary — what was built and why]

### Changed Files
[List of files with change type: added/modified/deleted]

### Platform Context

{{platform_context}}

If no platform context is provided, skip this section.

### Current Spec Slice

{{spec_slice}}

The current `.spec/spec.md` sections covering the modules this PR touches (Architecture, Data Model, API Surface, and any relevant Crosscutting Concepts) — the live system contract the change must fit. The spec is only patched into its ADDED/MODIFIED/REMOVED delta later, at `/sprint-refine`; at review time you check against the current spec, not a delta. If no spec is available (greenfield or pre-migration), skip this section.

### Quality Goals

{{quality_goals}}

The 3-5 quality attributes from `.brief/brief.md` (arc42 1.2), referenced here as guardrails — not reproduced in full. If no Brief is available, skip this section.

### Diff
[The relevant portion of the diff for this agent's focus area. For focused agents like silent-failure-hunter, include only files with error handling. For broad agents like code-quality-reviewer, include the full diff.]

### Instructions
Review ONLY what the PR changed. Do not flag:
- Pre-existing issues on unchanged lines
- Issues a linter, typechecker, or CI would catch
- Style or formatting preferences
- General observations that aren't actionable

Prioritise findings that diverge from the Current Spec Slice (code that contradicts the system contract the spec describes) or that violate one of the Brief's Quality Goals. Surface these first.

The Current Spec Slice and Quality Goals slots above are injected at dispatch for this review only — they are not reproduced durably in this template or stored on the PR.

For each finding, include:
- File path and line number
- Code snippet showing the issue
- Evidence explaining why this is a real issue
- Specific suggestion for fixing it

If no issues found, say so clearly.
