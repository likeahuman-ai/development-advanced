# codebase-explorer-prompt

Dispatched at sprint-plan 1.1.2 — the big discovery sweep. Launch the `codebase-explorer` agents in parallel; give each its own mode and the same seed below. The agent owns the method; this prompt only hands over what the session already holds.

Per dispatch, fill the `{…}` seed fields from the 1.1.1 conversation, then send:

> Explore this codebase for the sprint we're planning. Map the touched modules — scope, not an exhaustive sweep.
>
> - **Sprint intent:** {discussed intent — what this sprint is for}
> - **Touched modules (hints):** {module list from 1.1.1, e.g. `src/events`, `src/queue`, `src/deliver`, `src/api/webhooks` — a starting scope, expand as the code leads}
> - **Spec slice:** {if `.spec` exists → the relevant `.spec/spec.md` slice (see `spec-format`) — read it first. If none → omit this field.}
>
> Report your findings back for synthesis at the 1.1.3 discovery gate, where they are merged with the other explorers'.

Assign each parallel explorer a distinct mode — the agent's Architecture Mapping · Pattern Matching · Integration Analysis — or, for a small sprint, dispatch a single explorer with no mode set (its file says what it does then).
