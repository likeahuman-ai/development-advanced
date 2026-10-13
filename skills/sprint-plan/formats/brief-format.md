# Preamble

This file is the format of `.brief/brief.md`, the product pitch. One product, one brief. It is written like a CLAUDE.md at product level: the headings are for the AI, the words are the author's. `/sprint-plan` writes it once and edits it rarely; reviewers read its quality bars and its Definition of Done as part of the standard they judge against.

## Template

```markdown
# <Product name>

- **Date Last Edited:** YYYY-MM-DD

## Elevator pitch
<You pitch this product to a curious person in your coworking space, in the elevator. You have about one minute. They walk away with a basic understanding of: what kind of thing it is, what you are building, for whom, what want it serves, why it is not the same as what already exists, and what a day looks like once it works. Say it in concrete nouns. "We build robot greenhouses" beats "we reinvent agriculture". If a known product is close, lean on it: "Wikipedia, but inside a company". The minute is the only bound.>

## North star
<If this product is fully built, this is what is true. Write the dream, not the roadmap: the near goals, the stretch, and the things you only half believe. Say also what it will never be.>

## User roles
- **<user role>** — <what the system tells this role apart by>

## Quality bars
- **<bar>** — <one measurable bound>

## Definition of Done
- <a condition every landed ticket meets>
```

## Rules

- The headings are fixed: all present, in template order, none added. An anchor such as `.brief/brief.md#user-roles` never moves.
- **North star** is a picture, never a target or a date.
- **User roles** is a closed set, one line per role. A story names a role from this list and no other. A rename touches every story that names it.
- **Quality bars** holds the standing limits a change must not break: one bar per line, one bound per bar, such as "p95 under 200 ms on the production dataset". Reviewers judge a change against them.
- **Definition of Done** holds the conditions every landed ticket meets, beyond its own criteria and the build-order's checks. It never restates a quality bar: it points at it.
- Every edit comes with `Date Last Edited`.
- Everything else is a loose guardrail for the AI, and the author's intent wins: length, tone, depth are the author's.

## Ownership

`.brief/brief.md` is the source of truth. Artifacts reference a section by its anchor. Nobody quotes a section.
