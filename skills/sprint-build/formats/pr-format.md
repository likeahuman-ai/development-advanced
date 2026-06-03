# PR Description Template

Use this template when `/sprint-build` creates a pull request. Optimized for AI review.

## Title

Use a conventional/semantic title — `type(scope): summary` (`feat`/`fix`/`refactor`/`docs`/`chore`; append `!` to the type for a breaking change). `/sprint-build` derives `type` from the dominant ticket kind and `scope` from the touched subsystem. Example: `feat(auth): add session refresh`.

## Template

```markdown
## Summary
[1-3 sentences: what was built and why]

## Tickets
- Closes #[number] — [title] (US-###)
- Closes #[number] — [title] (US-###)
- Closes #[number] — [title] (US-###)

## Waves
- Wave 1: #[number], #[number] (parallel)
- Wave 2: #[number] (sequential)
- Wave 3: #[number], #[number] (parallel)

## Approach
[Brief strategy: what pattern was followed, key decisions made]

## Focus Areas
[Where reviewers should pay close attention]
- Security: [specific areas]
- Edge cases: [specific scenarios]
- Integration: [how new code connects to existing]
- Cross-wave interactions: [if later waves depend on earlier wave outputs, flag assumptions here]

## Risk Flags
- [ ] Breaking changes: [yes/no, details]
- [ ] Migration needed: [yes/no, details]
- [ ] Affects existing behavior: [yes/no, details]

## Skip List
[What reviewers should NOT flag]
- Generated files: [list if any]
- Lock files
- Formatting-only changes

## CI Coverage
[What automated checks already handle]
- TypeScript strict mode (typecheck)
- Pre-commit hooks (linting)

## Testing
- [What was tested]
- [What wasn't tested and why]
```

## Sizing Rules

- **Default:** One PR for all waves — the complete feature as a single reviewable unit.
- **> ~800 lines total:** Split at wave boundaries into stacked PRs. Each wave becomes its own PR.
- Each PR should be independently reviewable.

## Reviewer constraints (binding)

The **Skip List** and **CI Coverage** sections are not advisory — `/sprint-review` treats them as hard "do-NOT-flag" constraints: a reviewer must not raise a finding on anything listed in the Skip List or already covered by CI Coverage. This is how the producer (build) suppresses known false-positive sources for the consumer (review).

## Why This Structure

- Wave grouping shows the dependency structure and execution strategy
- Linked tickets help reviewers understand intent
- Focus areas prevent wasted time on low-risk code
- Cross-wave interaction flags help reviewers catch dependency assumptions
- CI coverage declaration reduces false positives (5-15% industry rate)
- Skip list prevents noise on generated/vendored code
- Risk flags help reviewers prioritize
