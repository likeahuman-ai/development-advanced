# codebase-explorer-prompt

Dispatched at sprint-tickets 2.1.1 — the fresh context sweep before tickets. Launch the `codebase-explorer` agents in parallel; give each its own mode and the same seed below. The agent owns the method; this prompt only hands over what the session already holds.

The `.spec` selects the shape of the dispatch:

**With `.spec`** — fill the `{…}` seed and send:

> Explore this codebase for the sprint we're ticketing. The `.spec` scopes where you start — bound the rest of your exploration to the sprint-touched paths below.
>
> - **Sprint scope:** {epics + the `US-###` slice + scope, from the 2.0 plan read — what's being sliced this sprint}
> - **Spec slice:** read the `.spec/spec.md` anchors that frame this sprint: {anchor pointers, e.g. `#api-surface`, `#data-model` — see `spec-format`; read the spec yourself, this only points}
> - **Stale paths (given — you cannot query history yourself):** {paths with commits newer than the spec's `last_updated`, computed this session from vcs log against that frontmatter}. Your tools are Read/Glob/Grep only, so you have no way to find these — they are handed to you. Re-explore them fresh against the code; scope the rest to the sprint-touched paths.
> - **Surface the shared seams:** call out any module two or more of the epics above touch — it feeds seam-ownership at 2.2.1.
>
> Report your findings back for synthesis at 2.1.2 Reconcile. Include any **spec mismatch** you hit — the spec's claim vs what the code actually does.

**Without `.spec`** — the full sweep. Fill the seed and send:

> Explore this codebase for the sprint we're ticketing — run all three modes. Map the touched modules; scope, not an exhaustive sweep.
>
> - **Sprint scope:** {epics + the `US-###` slice + scope, from the 2.0 plan read}
> - **Touched modules (hints):** {starting module list from the plan — expand as the code leads}
> - **Surface the shared seams:** any module two or more of the epics above touch — feeds seam-ownership at 2.2.1.
>
> Report your findings back for synthesis at 2.1.2 Reconcile.

Assign each parallel explorer a distinct mode (the agent's Architecture Mapping · Pattern Matching · Integration Analysis). On a small sprint one explorer running all three is fine.
