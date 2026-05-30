# Code Architect Prompt Template

Use this template when dispatching `code-architect` agents. Launch one per epic/feature from the PRD.

## Prompt

> Design the implementation for [EPIC/FEATURE NAME].
>
> ### PRD Section
> [Paste the relevant PRD section content]
>
> ### Codebase Exploration Findings
> [Paste relevant findings from the codebase-explorer agents]
>
> ### Architectural Decisions (ADRs)
> [Paste ADR content if `.adr/ADR.md` exists, otherwise omit this section]
> These are constraints from the planning phase. Treat the PRD section and ADRs as settled constraints — design within them. If an edge case conflicts with an ADR or the PRD, flag it rather than silently contradicting the decision.
>
> ### Your Job
> Produce:
> - **Write-set** — the exact file paths this ticket will touch, split into `creates` (new files) and `modifies` (existing files changed). This is the file-safety basis for parallel-wave grouping, so be precise and complete — a missed path can cause two parallel implementers to clobber the same file.
> - **depends-on** — artefacts this ticket needs to exist first, each marked HARD (won't compile/run without) or SOFT (works without, better with), referencing the producing ticket. Ordering only — distinct from the write-set.
> - **Verifiable requirements** — concrete, testable statements
> - **Acceptance criteria** — Given/When/Then, edge cases, verification commands
> - **Constraints** — files/patterns NOT to modify
> - **Dependencies** between tickets
> - **Complexity estimate** — S (single agent context, few files), M (full session, multiple files), L (multiple sessions, many systems)
>
> Complexity indicates AI resource cost, not human time.

## Usage

Replace `[EPIC/FEATURE NAME]` and paste the relevant PRD section and exploration findings. Each agent gets one epic or feature — don't overload a single agent with the entire PRD.
