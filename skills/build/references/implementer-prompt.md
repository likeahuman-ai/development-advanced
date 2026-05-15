# Implementer Prompt Template

Use this template when `/build` dispatches an implementer subagent for a ticket.

## Prompt

> ## Implement: [TICKET TITLE]
>
> ### Ticket
> [Paste full ticket body — objective, context, requirements, acceptance criteria, constraints, dependencies]
>
> ### Context
> This is ticket [N] of [TOTAL] in the build sequence. Previous tickets already completed: [list titles]. The codebase reflects all prior work.
>
> ### Before You Begin
> If you have questions about the requirements, constraints, or approach — ask them now before starting work. It's better to clarify upfront than to build the wrong thing.
>
> ### Coding Standards
>
> {{coding_standards}}
>
> If coding standards are provided above, follow them. They reflect the participant's own conventions. If the slot is empty, follow existing codebase patterns only.
>
> ### Your Job
> 1. **Implement** the ticket spec exactly. Follow existing codebase patterns and any coding standards above.
> 2. **Write tests** if the ticket includes test-related acceptance criteria.
> 3. **Verify** — run any verification commands from the acceptance criteria (`pnpm test`, `pnpm typecheck`, etc.).
> 4. **Commit** — granular commits per logical unit. Good commit messages.
>    - If a commit fails (pre-commit hook, lint, formatting), fix the issue and retry ONCE. If the second commit also fails, report BLOCKED with the exact error. Do not retry further.
> 5. **Self-review** — before reporting, review your own work:
>    - Did you implement everything in Requirements?
>    - Did you meet all Acceptance Criteria?
>    - Did you respect all Constraints?
>    - Did you overbuild anything not requested?
> 6. **Report** your status:
>    - **DONE** — all requirements met, tests pass, self-review clean
>    - **DONE_WITH_CONCERNS** — implemented but [specific concerns]
>    - **NEEDS_CONTEXT** — need clarification on [specific question]
>    - **BLOCKED** — cannot proceed because [specific blocker]

## Model Selection

- **S** (small) → sonnet
- **M** (medium) → sonnet
- **L** (large) → inherit (Opus)

## Handling Results

- **DONE** → proceed to spec review
- **DONE_WITH_CONCERNS** → assess concerns, then proceed to spec review
- **NEEDS_CONTEXT** → provide context, re-dispatch same model
- **BLOCKED** → provide more context, escalate model, break ticket down, or escalate to user
