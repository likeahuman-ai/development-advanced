# Build-Order Format

The build-order is a single GitHub issue (label `build-order` + version) that `/build` executes **without re-reading the tickets**. It carries everything the build orchestrator needs to plan and dispatch; the tickets carry everything the implementer needs to build. Build reading a ticket to brief an implementer is correct; build reading a ticket to recompute the *plan* is the bug this format prevents.

## Template

```markdown
Build order for **Epic #[N]** ([feature/version]). Plan status: **APPROVED — authoritative.**

## Scope
Build all [N] tickets. #[X] is stretch → build it unless told otherwise.   ← a DECISION, not an option

## Verify   (run exactly this at each wave boundary — nothing else)
[the exact command, e.g. pnpm -F @pkg typecheck && pnpm -F @pkg test]
# note any suites deferred to deployment; do NOT run workspace-wide.

## Tickets   (recorded basis for the waves — see Authority)
#[N] [S/M/L] [title]
    depends-on: [#producer (HARD|SOFT), or —]
    creates:    [new file paths, or —]
    modifies:   [existing file paths, or —]

## Parallel Waves   (HARD-dep-free AND file-disjoint — dispatch together, no pre-wave checks)
Wave 1: #a, #b, #c        (disjoint write-sets)
Wave 2: #d                 (modifies a file #a touches — waits for Wave 1)
Parallelism: [X] of [Y] tickets parallel across [W] waves.

## Build Sequence   (sequential fallback — fundamental /build reads this; advanced ignores it)
1. #a [S/M/L] — [title]
2. ...

## PR Grouping   (coupling, not line count)
PR A: #a, #d  — [shared runtime boundary]
PR B: #b, #c  — [...]
```

## Field reference

| Field | Read by | Purpose |
|-------|---------|---------|
| **Plan status: APPROVED** | build | This is the approved plan, not a draft. Build executes; it does not re-gate. |
| **Scope** | build | The decision of what to build, incl. stretch defaults. Closes the option-not-decision tax. |
| **Verify** | build | The exact command per wave boundary. Removes "decide the command yourself." |
| **Tickets** (`depends-on`, `creates`, `modifies`) | tickets→build | `depends-on` = HARD ordering. `creates ∪ modifies` = write-set (file-safety). |
| **Parallel Waves** | advanced build | The **authoritative** execution plan. Each wave is HARD-dep-free and file-disjoint by construction. |
| **Build Sequence** | fundamental build | Sequential fallback (collision-safe by being one-at-a-time). |
| **PR Grouping** | build Phase 3 | Which tickets share a PR (by coupling). |

**Not present, by design:** base branch / placement (the window/extension owns that, not the plugin); full requirements/acceptance (stay in the ticket, read at dispatch); business justification (in the PRD).

## Authority rule

The write-sets are recorded **for completeness and audit** — a reader can see *why* the waves are grouped as they are. But the `## Parallel Waves` section is **AUTHORITATIVE**: build dispatches the waves as written and MUST NOT recompute or re-verify disjointness from the write-sets. Re-checking a correct grouping is a check whose normal outcome is "confirmed" — noise. Trust the plan; act only on a *real* failure (the commit-time residual guard).

## Wave computation (how `/tickets` produces the waves)

Inputs per ticket: `depends-on` (HARD only; SOFT does not constrain order) and `write-set = creates ∪ modifies` (exact paths; a shared *directory* is not a conflict, a shared *file* is).

```
completed = ∅ ; waves = [] ; remaining = all tickets
while remaining:
    ready = { t ∈ remaining : t.depends-on ⊆ completed }      # HARD deps satisfied
    if ready == ∅: ERROR "HARD dependency cycle" → flag to user (reclassify one HARD→SOFT)
    wave = [] ; used = ∅
    for t in sort(ready, by priority[blocker>important>nice], then id):
        if t.write-set ∩ used == ∅:                           # file-disjoint with this wave
            wave.append(t) ; used ∪= t.write-set
    waves.append(wave) ; completed ∪= wave ; remaining −= wave
```

**Accepted trade-off:** file-disjoint waves can be narrower than HARD-dep-only waves (two dep-independent tickets that share a file split across waves). Under the shared-tree constraint this is the correct price: determinism + no silent clobber > peak parallelism.
