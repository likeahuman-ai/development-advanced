# Preamble

This file is the format of the build-order: one GitHub issue per sprint that shows people how the sprint's tickets build, wave by wave, and holds the commands that check every build. `/sprint-tickets` writes it. `/sprint-build` and `/sprint-refine` read only its `## Verify`: Build takes the build order from GitHub's blocked-by edges, so the waves here are a view, never an input.

## File

- **Location:** One GitHub issue per sprint, titled `Build order — sprint v{N}`, with the labels `build-order` and `v{N}`. Nothing lives in a dotdir.
- **Shape:** The body is flat `## ` sections, in template order, all present, no prose before the first. A parser uses `## ` to find each section.

## Template

```markdown
## Parallel Waves

### Wave 1
- #N <title> — blocked by: —

### Wave 2
- #N <title> — blocked by: #N[ #N]

## Verify
- provision: `<command>`
- check: `<command>`[, `<command>`]
```

One entry per slot:

| Slot | Rule |
|---|---|
| **Parallel Waves** | Mandatory. A wave is every ticket whose blockers all sit in earlier waves; the tickets of one wave build in parallel. One line per ticket: its number, its title as issued, its blockers. Derived from the tickets' `Blocked by` lines; never a fact of its own. |
| **provision** | Mandatory. One real command that installs the project's dependencies, read off the repository's tooling, such as `pnpm install --frozen-lockfile`. A new worktree has no dependencies and runs it first. |
| **check** | Mandatory. The repository's own commands that must pass on every ticket and before every land: typecheck, lint and test, those the repository has, read off its scripts. Never invented, never a placeholder. A ticket's own `test:` line is not listed here: the ticket owns it. |

## Mutability

- **Parallel Waves** is rewritten whenever a ticket or a blocker changes before the build.
- **Verify** changes only when the repository's tooling changes.

## Ownership

The tickets own their blockers; this issue shows them. It names a ticket by number and title only, and never quotes a ticket body. `## Verify` is the one home of the project-wide check: an implementer, a fix agent and the land run it as written.

## Example

```markdown
## Parallel Waves

### Wave 1
- #41 Store auth sessions in Convex — blocked by: —
- #42 Configure login rate limits — blocked by: —

### Wave 2
- #43 Add the login handler — blocked by: #41 #42
- #44 Add the logout handler — blocked by: #41

### Wave 3
- #45 Show the session banner — blocked by: #43 #44

## Verify
- provision: `pnpm install --frozen-lockfile`
- check: `pnpm typecheck`, `pnpm lint`, `pnpm test`
```
