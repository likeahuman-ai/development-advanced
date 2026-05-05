# Design Review Prompt Template

Use this template when `/review` dispatches the `design-reviewer` agent.

## Prompt

> ## Design Review: [PR TITLE]
>
> ### PR Diff
> [Paste the relevant diff — CSS, Tailwind config, component files with JSX/styling]
>
> ### Preferred Tech Stack
> Convex + Next.js + Tailwind CSS + Storybook. If the PRD approves a different stack, that takes precedence.
>
> ### Visual Direction from PRD
> [Paste the Visual Direction section from the PRD, or "Not available — skip PRD conformance checks and run anti-slop scan only."]
>
> ### Your Job
> Run all three checks:
> 1. **Tech stack conformance** — is the code using the preferred stack (or the approved alternative)?
> 2. **PRD conformance** — do fonts, colours, theme, and layout match the Visual Direction? (Skip if not available.)
> 3. **Anti-slop scan** — check for these 10 patterns:
>    - AI default fonts (Inter, Roboto, Open Sans, Arial, system-ui)
>    - Purple-to-blue gradients
>    - Nested card components
>    - Identical repeating card grids
>    - Side-stripe borders (border-left/right > 1px)
>    - Gradient text (background-clip: text)
>    - Uniform spacing everywhere
>    - Pure black (#000) or white (#fff)
>    - Centre-everything layouts
>    - Dark mode with neon/glowing accents
>
> Report each finding with: confidence (0-100), severity (high/medium/low), file:line location, what you found, why it matters, and a fix suggestion.
>
> If a check produces zero findings, say so briefly. Don't manufacture issues.

## Usage

Replace `[PR TITLE]` with the PR title. Paste the diff and Visual Direction section. The main model handles extracting the Visual Direction from the PRD before dispatching.
