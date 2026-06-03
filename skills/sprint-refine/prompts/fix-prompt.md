# Fix Dispatch Prompt Template

Use this template when `/sprint-refine` dispatches an implementer subagent for a review finding.

## Prompt

> ## Fix: {FINDING TITLE}
>
> ### Finding
> **File:** {file_path}:{line_number}
> **Agent:** {agent_name} (confidence: {score}/100)
> **Description:** {finding description}
> **Suggested fix:** {suggested fix from review}
>
> ### Code Context
> {Paste the relevant code — the file content around the finding, enough for the implementer to understand the surrounding logic. Typically 20-40 lines centered on the finding.}
>
> ### Spec slice (optional, read-only context)
> {{spec_slice}}
>
> Include this slot ONLY when the fix touches a module documented in `.spec/spec.md` — paste the matching spec section(s) so the implementer keeps the change consistent with the documented contract. Like `/sprint-build`, this enrichment is pasted fresh at dispatch and is never stored anywhere; it is read-only context — do NOT edit the spec here (the spec delta is applied later, in the `/sprint-refine` skill, after the fix lands). If the slot is empty or absent, ignore it.
>
> ### Your Job
> The finding was already verified and scored — implement the fix; do not re-litigate whether the finding is valid.
> 1. **Read** the finding and understand what needs to change.
> 2. **Implement** the fix. Follow existing codebase patterns.
> 3. **Verify** — run the build-order's single authoritative `## Verify` command, **scoped to the package you changed** (e.g. `pnpm --filter <package> test` / `typecheck`). Do NOT run workspace-wide checks. If no build-order is in play, run the relevant package-scoped check the finding implies.
> 4. **Self-review** — before reporting:
>    - Did you fix the specific issue described?
>    - Did you avoid changing unrelated code?
>    - Did you introduce any new issues?
> 5. **Report** your status:
>    - **DONE** — fix applied, verified
>    - **DONE_WITH_CONCERNS** — fixed but {specific concern}
>    - **NEEDS_CONTEXT** — need clarification on {specific question}
>    - **BLOCKED** — cannot fix because {specific reason}

## Model Selection

Always sonnet. Review fixes are scoped and small.

## Handling Results

- **DONE** → proceed to next finding
- **DONE_WITH_CONCERNS** → assess concerns, note for user, proceed
- **NEEDS_CONTEXT** → provide context from the PR/codebase, re-dispatch
- **BLOCKED** → skip this finding, report to user as "could not auto-fix"
