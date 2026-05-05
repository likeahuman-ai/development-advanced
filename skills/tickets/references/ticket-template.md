# GitHub Issue Body Template

Use this template for every ticket created by `/tickets`.

## Template

```markdown
## Objective
[One sentence: what to build and why]

## Context
- Relevant files: `src/path/to/file.ts`, `src/path/to/other.ts`
- Current behavior: [what happens now]
- Expected behavior: [what should happen after]

## Requirements
- [ ] Concrete, verifiable requirement 1
- [ ] Concrete, verifiable requirement 2
- [ ] Concrete, verifiable requirement 3

## Acceptance Criteria
- Given [state], when [action], then [result]
- Edge case: [scenario] → [expected handling]
- Run `pnpm test` — all pass
- Run `pnpm typecheck` — no errors

## Constraints
- Do NOT modify: [off-limits files/directories]
- Must use: [specific patterns, libraries]
- Must follow: CLAUDE.md conventions

## Dependencies
- Blocked by: #[issue number]
- Blocks: #[issue number]
```

## Principles

- One ticket = one independently verifiable change = roughly one PR
- Include verification commands so the implementing agent can self-check
- No business justification (that's in the PRD)
- No CLAUDE.md duplication — tickets contain only the delta specific to this task
- Acceptance criteria are testable, not subjective
