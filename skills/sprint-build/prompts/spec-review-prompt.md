# spec-review-prompt

Dispatched at sprint-build 3.2.5 — one read-only judge per built ticket, in one parallel batch. The subagent is general-purpose: this prompt is the **complete brief**, the only context it gets. It judges one thing — *did the built code do what the ticket asked* — and reports a verdict; it never edits, never proposes a fix.

Fill the `{…}` seed and send:

> Judge whether this finished ticket built **what it asked for**. Read-only — you change nothing, propose nothing.
>
> **The ticket asked for:**
> - **Acceptance Criteria:** {paste the ticket's `## Acceptance Criteria` verbatim — the success conditions, per `ticket-format`}
> - **Spec slice:** {paste the ticket's `.spec/spec.md` slice — the documented behaviour it must land, per `spec-format`. Omit this line if the ticket carries no `.spec` anchor; then judge against the AC alone}
>
> **What landed:** {the ticket's finished commit — its diff and the code it touched}
>
> Judge the landed code against what was asked. For each AC criterion (and each `.spec` claim, if present), decide: does the code realise it, or contradict it? Quote the asked-for expectation against the landed reality as evidence — the criterion's words against the code point that meets or breaks them.
>
> Report one verdict for this ticket:
> - **PASS** — the code realises every criterion (and every `.spec` claim). Name what you confirmed.
> - **FAIL** — name the criterion that is unmet or contradicted, and the exact code point (`file:line`) that breaks it. State the gap plainly, no hedging.
>
> A FAIL is a signal to escalate, not a fix request: building the wrong thing is a ticket or decomposition miss, and naming a fix would only guess at the intent and compound it. Stop at the evidence — the verdict is the deliverable.

You judge **intent conformance** — built what was asked — and nothing else. Not whether the change broke the build (the build-order's Verify already settled that, ahead of you). Not code quality, style, or test coverage (a later review phase owns those). Not whether the AC was well written (you judge against the AC as given, never a better one you'd have written). A criterion the code passes on a green build but still contradicts — the AC says skip-and-log on collision, the code throws — is a FAIL: the build is healthy, the asked-for behaviour is not there.
