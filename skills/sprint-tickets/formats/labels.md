# Label Taxonomy

Single source of truth for every GitHub label the development-advanced cycle uses. `/sprint-tickets` ensures the **standing set** exists before it creates any issue. A missing standing label silently fails the `--add-label` that drives the cycle state machine, so this set must exist on every repo — including a fresh one `/sprint-tickets` just created via `gh repo create`. (Deferred review findings do **not** become labelled Issues — they go to the findings backlog `.sprint/backlog.md`; see `/sprint-review`.)

## Standing set (identical every cycle — ensured by `/sprint-tickets`)

| Label | Colour | Meaning |
|-------|--------|---------|
| `complexity/S` | `C2E0C6` | Small — AI resource cost (never time) |
| `complexity/M` | `FEF2C0` | Medium — AI resource cost |
| `complexity/L` | `F9D0C4` | Large — AI resource cost |
| `build-order` | `5319E7` | The build-order plan issue (pinned while `/sprint-build` runs) |
| `needs-review` | `FBCA04` | PR built, awaiting `/sprint-review` |
| `needs-refine` | `D93F0B` | PR reviewed, awaiting `/sprint-refine` |
| `cycle-complete` | `0E8A16` | Cycle closed — `/sprint-refine` DoD gate passed |

There is **no `priority/*` family**. Priority never gated whether a ticket ships — every ticket in an approved sprint gets built — so it carried no state. Wave-fill ties break on issue number, not priority.

### Ensure block (run once per cycle, idempotent)

`--force` creates the label or updates it in place, so re-running is a no-op on an established repo and seeds the whole machine on a fresh one. Run the block in a single Bash call:

```bash
gh label create "complexity/S"   --color C2E0C6 --description "Small — AI resource cost" --force
gh label create "complexity/M"   --color FEF2C0 --description "Medium — AI resource cost" --force
gh label create "complexity/L"   --color F9D0C4 --description "Large — AI resource cost" --force
gh label create "build-order"    --color 5319E7 --description "Build-order plan issue (pinned during build)" --force
gh label create "needs-review"   --color FBCA04 --description "PR built, awaiting /sprint-review" --force
gh label create "needs-refine"   --color D93F0B --description "PR reviewed, awaiting /sprint-refine" --force
gh label create "cycle-complete" --color 0E8A16 --description "Cycle closed — DoD gate passed" --force
```

## Per-cycle set (created by `/sprint-tickets` each cycle)

| Label | Colour | Meaning |
|-------|--------|---------|
| `v{N}` | `0052CC` | Cycle version — derived from `.sprint/sprint-v{N}.md` |
| `epic:{name}` | `6F42C1` | Epic grouping (hierarchical breakdowns) |
| `feature:{name}` | `BFD4F2` | Feature grouping (hierarchical breakdowns) |

`v{N}` derives from the Sprint Plan filename. Hierarchy labels use the real epic/feature names from the breakdown. Create with `--force` so a re-run is idempotent.

**Slug rule for `epic:`/`feature:`:** lowercase, spaces → hyphens, strip other punctuation — so the same epic always produces the same token across phases (e.g. `Onboarding & Setup` → `epic:onboarding-setup`).

## Cycle state machine (which labels carry forward state)

The three cycle-enforcement labels move a PR through the pipeline — exactly one is set at a time:

```
/sprint-build   → needs-review     (PR created as draft + needs-review)
/sprint-review  → needs-refine     (remove needs-review, add needs-refine; draft → ready)
/sprint-refine  → cycle-complete   (remove needs-refine, add cycle-complete)
```

Each transition is a **single atomic edit** — `gh issue edit [n] --remove-label OLD --add-label NEW` — so a PR never carries two state labels at once.

The Sprint Plan's frontmatter `status:` tracks the coarse cycle milestone in parallel (`draft → built → archived`); `/sprint-build` flips it to `built` when the code is done. `cycle-complete` lives only on the PR — it is not duplicated as a sprint status.

Issues are open→closed only (closed by `/sprint-build` at wave completion; never reopened). The build-order issue is pinned by `/sprint-tickets`, then unpinned **and** closed by `/sprint-build`.
