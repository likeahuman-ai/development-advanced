# Preamble

This file is the format of `.sprint/sprint-v{N}.md`: what one sprint delivers and why. `/sprint-plan` writes it and lands it in the plan PR. `/sprint-tickets` reads it as its whole brief, with no person in the loop, so every decision Tickets needs is made here. The next `/sprint-plan` run sets its status once the sprint has landed.

## File

- **Location:** `.sprint/sprint-v{N}.md`, one file per sprint.
- **Shape:** YAML frontmatter, then a `# ` title, then flat `## ` sections in template order. Optional sections are absent when empty, never left as a stub.
- **Numbering:** `v{N}` is sequential from `v1`, with no gap. It names the sprint's label `v{N}` and the plan branch `docs/sprint-v{N}`.

## Template

```markdown
---
version: N
status: draft
date: YYYY-MM-DD
author: <name>
previous: sprint-v{N-1}.md | null
---

# Sprint v{N} — <title>

## Goal
This sprint succeeds when <one objective>.

## Solution
<one plain paragraph: what changes for the people who use the product>

## Stories
- US-### — <the part of the story this sprint delivers>

## Epics
- `epic:<name>` — <the stories and scope rows it groups>

## Scope
| In scope | Out of scope |
|---|---|
| <item> | <item> |

[## Architecture
<two or three lines: how the sprint changes the system, with .spec/spec.md#<anchor> and ADR-### pointers>]

[## Risks
- <risk> — <what it can break>]
```

One entry per slot:

| Slot | Rule |
|---|---|
| **version** | Mandatory. The integer `N`. |
| **status** | Mandatory. `draft` while the sprint is planned and built. `built` when every ticket of the sprint has landed; the next plan sets it, beside the spec sweep. `archived` when a newer plan exists; the next plan sets it. `abandoned` when the user drops a draft. At most one `draft` exists in `.sprint/`. |
| **date** | Mandatory. The day the plan was written. Never edited. |
| **author** | Mandatory. The person who owns the sprint. Never invented. |
| **previous** | Mandatory. The prior plan's file name, `null` for `v1`. |
| **Goal** | Mandatory. One sentence, pass or fail. An "and" in the objective means two sprints: split or cut. At most three conditions, as a list under the sentence, when they genuinely pass or fail together. |
| **Solution** | Mandatory. One paragraph without file names or jargon, for a person who does not read code. |
| **Stories** | Mandatory. One line per `US-###`, with the part this sprint delivers. Never the story's text. |
| **Epics** | Mandatory. One line per epic, `epic:<name>` in lower case `a–z 0–9 -`, with what it groups. Tickets uses each token as written, as a label. A sprint with one epic still names it. |
| **Scope** | Mandatory. One table, both columns filled. The only home of what is out. |
| **Architecture** | Optional. The shape of the change only. A decision is cited by its real `ADR-###`, never by an invented number. No directory trees or component lists: `.spec/spec.md` holds those. |
| **Risks** | Optional. One line per real risk. |

## Mutability

- **status** changes as the slot says, in the commit that makes it true. Nothing else changes once the plan PR lands. A changed plan is a new version.

## Ownership

The plan is the source of truth for the sprint's goal, scope and epic partition. It references `US-###`, `ADR-###` and `.spec/spec.md` anchors and never copies them. Progress is a GitHub fact, never a line here.

## Example

```markdown
---
version: 3
status: draft
date: 2026-10-13
author: tim
previous: sprint-v2.md
---

# Sprint v3 — Webhook delivery retries

## Goal
This sprint succeeds when a failed webhook dispatch is retried until it is delivered or set aside after two hours.

## Solution
Failed deliveries stop disappearing silently: the platform retries them on its own and sets aside the ones that keep failing, so integrators receive every event without anyone watching a queue.

## Stories
- US-007 — automatic retry with backoff; the notification email ships next sprint
- US-012 — the dead-letter store, persisted and queryable

## Epics
- `epic:retry-pipeline` — US-007; retry, backoff and the delivery-outcome counter
- `epic:dead-letter-store` — US-012; dead-letter persistence

## Scope
| In scope | Out of scope |
|---|---|
| Retry with exponential backoff | Manual replay from the dead-letter store |
| Dead-letter persistence after the last attempt | The dead-letter notification email |
| Delivery-outcome counter | Multi-region dispatch |

## Architecture
A retry queue sits between the dispatcher and the worker pool (`.spec/spec.md#runtime--data-flow-view`). Dead letters use the existing store (ADR-004); no schema change.
```
