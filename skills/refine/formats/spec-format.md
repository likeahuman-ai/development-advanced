# Spec Format Reference

The spec is a living system description. It always reflects what EXISTS now — never aspirational state. Updated after every build cycle by the spec-writer agent.

## Template

```markdown
# {Project Name} — Spec

Last updated: {date} (after PRD v{N} cycle)

## Architecture

{Components, how they connect, deployment shape. Keep to 1 paragraph + a list.}

## Stack

{Core technologies with version constraints. Reference ADRs for "why":
- Next.js 16 (app router) — see ADR-001
- Convex (backend + real-time) — see ADR-003
- Tailwind CSS v4
}

## Data Model

{Key schemas and relationships. Use type notation, not prose:
- `users`: id, email, name, role, createdAt
- `workshops`: id, title, slug, status, instructorId → users.id
}

## API Surface

{Endpoints and contracts. Group by domain:

### /api/workshops
- GET /api/workshops — list all (paginated)
- POST /api/workshops — create (body: { title, slug })

### Convex Functions
- `getWorkshop(id)` → Workshop | null
- `listParticipants(workshopId)` → Participant[]
}

## Key Patterns

{Established patterns in the codebase:
- Mutations use server actions (not API routes)
- All queries use Convex reactive subscriptions
- Error boundaries at route segment level
- Form validation with Zod schemas
}

## Directory Structure

{Where things live:
- `src/app/` — Next.js pages and layouts
- `src/components/` — domain components (auth/, workshop/, settings/)
- `convex/` — backend functions and schema
- `packages/dls/` — design system components
}

## Infrastructure

{Deploy target and services:
- Vercel (frontend + serverless)
- Convex Cloud (backend)
- Clerk (auth)
- Brevo (email)
}
```

## Update Strategy

### When spec exists (update mode)

The spec-writer receives: existing spec + PR diff + full files touched by diff.

Rules:
1. **Only modify sections affected by the diff.** If the diff touches API routes, update API Surface. If it adds a new dependency, update Stack. Leave unaffected sections ALONE.
2. **Add, don't re-describe.** If a new endpoint was added, add it to the list. Don't rewrite the section.
3. **Remove when removed.** If the diff removes a capability (deleted file, removed endpoint), remove it from the spec.
4. **Never describe aspirational state.** The PRD says what will be built. The spec says what IS built. If the PRD planned 5 endpoints but only 3 were built, the spec lists 3.

### When no spec exists (creation mode)

The spec-writer receives: codebase explorer results + PRD + ADRs.

Rules:
1. Fill all 7 sections from the codebase context.
2. Use types and lists, not paragraphs.
3. Target 100-150 lines for initial creation.
4. Every Stack entry should reference an ADR if one exists.

## Drift Detection

When updating an existing spec, compare:
1. **Directory Structure** section vs actual directory listing. If new top-level dirs appeared (not in the diff), flag and investigate.
2. **Stack** section vs `package.json` dependencies. If deps were added/removed outside the diff, update.
3. **Architecture** section vs actual component boundaries. If a new service or package appeared, update.

If drift is detected: read the relevant new files, update the spec, and note in the commit message what drift was caught.

## Level of Detail

- **Types > prose.** Show the interface definition, not a paragraph describing it.
- **One level of nesting max.** No sub-sub-sections within spec sections.
- **Target: 100-200 lines.** If longer, the project grew significantly — still keep it concise.
- **Reference, don't duplicate.** Point to ADRs for "why." Point to files for implementation. The spec is a map, not the territory.

## What NOT to Include

- Implementation details (how a function works internally)
- Test descriptions or test file paths
- Build/CI/CD configuration
- Git workflow or branching strategy
- Environment variable values (just names)
- Comments in code or documentation standards
