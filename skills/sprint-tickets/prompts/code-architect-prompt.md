# code-architect-prompt

Prepared at sprint-tickets 2.2.1, dispatched at 2.2.2 — one `code-architect` per epic/feature, in parallel. The agent owns the design method and the seam discipline; this prompt only hands over the slice of 2.0 + 2.1 the session already holds. Pointers + boundaries, not a code dump.

Fill the `{…}` seed and send:

> Design the tickets for the **{epic title}** epic.
>
> - **Epic scope:** {the epic's `.sprint` detail + the `US-###` slice it serves + the success metric — from the 2.0 plan read}
> - **Governing ADR Y-statement(s):** {paste the Y-statement verbatim for each `.adr` record that governs this epic — the blockquote line only, per `adr-format`; the Y-statement is the only part lifted verbatim, IDs travel as `ADR-###` pointers}
> - **Module map:** {the modules this epic touches, from 2.1.1's exploration}
> - **Shared-seam assignments:** {name each shared seam this epic touches and its single owner — e.g. `payment-gateway` → owned by the "Payment setup" epic; omit if this epic touches none}
> - **Spec slice:** {the `.spec/spec.md` anchors for the touched modules — pointers only, e.g. `#api-surface`, `#crosscutting-concepts--patterns`; read the spec yourself, per `spec-format`. Omit if no `.spec`}
>
> Report your ticket-sized units back for consolidation at 2.3.1 — the design lives in your report, no design-doc file.

A seam owned elsewhere is named as data here; the agent applies its own rule (consume as given, flag don't fork). Greenfield or pre-migration: drop the ADR and Spec lines — the agent designs from the epic scope and the code alone.
