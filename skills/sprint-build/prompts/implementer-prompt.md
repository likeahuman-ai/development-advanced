# Implementer Prompt Template

Use this template when `/sprint-build` dispatches an implementer subagent for a ticket.

## Prompt

> ## Implement: [TICKET TITLE]
>
> ### Ticket
> [Paste full ticket body — objective, context, requirements, acceptance criteria, constraints, dependencies]
>
> ### Context
> This is Wave [W] of [TOTAL_WAVES]. You are implementing ticket [M] of [TOTAL_IN_WAVE] in this wave. All prior waves are committed and visible in the working tree. Do NOT assume other tickets in this wave are complete — they may be executing in parallel. Prior waves completed: [list wave summaries with ticket titles].
>
> ### Spec slice (read-only context)
> {{spec_slice}}
>
> ### Governing ADR (if any)
> {{governing_adr}}
>
> **Note — enrichment reproduced inline here on purpose.** This dispatch payload is the *one exception* to the artifact "reference, don't reproduce" rule. The spec slice and ADR Y-statement above are pasted in full because this prompt is ephemeral — the orchestrator enriches it fresh at dispatch and it is NEVER stored in the GitHub Issue. The reference-not-reproduce rule governs durable artifacts, not this transient dispatch prompt. If either slot is empty, ignore it.
>
> ### Authority: ticket is the plan, spec is context
> The **ticket is the plan to implement** — execute it exactly as written. Do not re-design, re-scope, or re-confirm it. Start immediately. The **spec slice** above is read-only context that describes the surrounding system; do NOT edit the spec here (spec updates happen later, at `/sprint-refine`). The **ADR Y-statement** above is a binding architectural constraint — honour it, do not contradict it. Stop only if faithful execution is *impossible*: a file, symbol, or dependency the ticket references does not exist, or two requirements directly contradict — then report NEEDS_CONTEXT with the specific blocker. A question whose answer is "yes, as the ticket says" is noise; don't ask it.
>
> ### Coding Standards
>
> {{coding_standards}}
>
> If coding standards are provided above, follow them. They reflect the user's own conventions. If the slot is empty, follow existing codebase patterns only.
>
> ### Your Job
> 1. **Implement** the ticket spec exactly. Follow existing codebase patterns and any coding standards above.
> 2. **Write tests** if the ticket includes test-related acceptance criteria.
> 3. **Verify** — run the verification commands from the build-order's single authoritative `## Verify` section (tickets no longer carry their own `pnpm` commands), **scoped to the package you changed** (e.g. `pnpm --filter <package> test`, `pnpm --filter <package> typecheck`, or `pnpm --filter <package> exec tsc --noEmit`). Do NOT run workspace-wide checks: in a parallel wave, sibling tickets are mutating other packages at the same time, so a root-level `pnpm test`/`typecheck` would be both slow and unreliable. The orchestrator runs one full-workspace verification at the wave boundary.
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
