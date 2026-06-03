# Spec Format Reference

The spec is a living system description. It always reflects what EXISTS now — never aspirational state. It is **cycle-living**: patched by a lightweight delta after every build cycle by the spec-writer agent.

## Template

```markdown
# {Project Name} — Spec

Last updated: {date} (after sprint-v{N} cycle)

## Architecture

{Components, how they connect, deployment shape. Keep to 1 paragraph + a list.}

## Runtime / Data-flow view

{How a request/event moves through the system at runtime — the dynamic view, first-class alongside the static Architecture. One or two key flows, as a numbered sequence:
1. Request hits Vercel edge → Next.js route handler
2. Server action calls Convex mutation → reactive subscription updates client
3. Webhook (Clerk) → Convex HTTP action → user record sync
}

## Data Model

{Key schemas and relationships. Use type notation, not prose:
- `users`: id, email, name, role, createdAt
- `projects`: id, title, slug, status, ownerId → users.id
}

## API Surface

{Endpoints and contracts. Group by domain:

### /api/projects
- GET /api/projects — list all (paginated)
- POST /api/projects — create (body: { title, slug })

### Convex Functions
- `getProject(id)` → Project | null
- `listMembers(projectId)` → Member[]
}

## Crosscutting Concepts & Patterns

{Established patterns that cut across the codebase. ENUMERATE the concerns, not just a flat list of conventions:

### Auth & security
- Clerk session enforced in Convex via `ctx.auth.getUserIdentity()`
- Role checks centralized in `lib/authz.ts`; admin routes gated server-side

### Error handling
- Error boundaries at route segment level
- Mutations throw typed `ConvexError`; client maps to toast

### Logging & observability
- Structured logs via `lib/log.ts`; request-scoped correlation id
- Convex function metrics surfaced in dashboard

### Glossary & naming
- "Project" = a tracked initiative; "Task" = one unit of work within it
- Convex functions: `verbNoun` (e.g. `listMembers`); components: PascalCase
}

## Stack

{Core technologies with version constraints. Kept in-spec; reference ADRs for "why":
- Next.js 16 (app router) — see ADR-001
- Convex (backend + real-time) — see ADR-003
- Tailwind CSS v4
}

## Directory pointer-map

{A SHORT pointer map — where to look, not a full tree. One line per significant area:
- `src/app/` — Next.js pages and layouts
- `src/components/` — domain components (auth/, projects/, settings/)
- `convex/` — backend functions and schema
- `packages/dls/` — design system components
}

## Infrastructure

{Deploy target and services:
- Vercel (frontend + serverless)
- Convex Cloud (backend)
- Clerk (auth)
- Brevo (email)

Integrations (the system's edges):
- Outbound — services this calls (e.g. Stripe API, Brevo)
- Inbound — webhooks / identity it receives (e.g. Clerk webhook, Stripe webhook)
}

## Constraints

{Standing, project-wide hard rules an implementer must honour (per-ticket write-sets scope the rest):
- **Never touch** — globs that must not be modified (e.g. `legacy/**`, generated files)
- **Ask first** — areas that need confirmation before changing (e.g. `convex/schema.ts`)
}
```

## Update Strategy

### Delta-patch model (PRIMARY — every cycle with an existing spec)

The spec is patched by a **lightweight delta**, not rewritten. At `/sprint-refine` the spec-writer receives: existing spec + PR diff + full files touched by diff, then **emits and applies** a set of change hunks.

Rules:
1. **Emit ADDED / MODIFIED / REMOVED hunks.** Each hunk names the target section (and heading anchor) and describes the surgical change. Apply them directly to the spec.
2. **Touch only changed sections.** If the diff touches API routes, patch `## API Surface`. If it adds a dependency, patch `## Stack`. Leave every unaffected section byte-for-byte ALONE.
3. **Add, don't re-describe.** A new endpoint is appended to the list — the surrounding prose is not rewritten.
4. **Remove when removed.** A deleted file / removed endpoint is deleted from the spec.
5. **Git is the archive.** Do NOT keep a `changes/` or `archive/` tree, a changelog section, or any in-file history. The previous spec state lives in git; `git log -p .spec/spec.md` is the history.
6. **Never describe aspirational state.** The Sprint Plan says what will be built. The spec says what IS built. If the Sprint Plan planned 5 endpoints but only 3 were built, the spec lists 3.

Free-hand full rewrites are NOT a normal-cycle operation — they are reserved for creation mode only.

### Creation mode (first cycle, no spec exists)

This is the ONLY mode where the spec-writer writes free-hand / full prose. It receives: codebase explorer results + Sprint Plan + ADRs.

Rules:
1. Fill all sections from the codebase context.
2. Use types and lists, not paragraphs.
3. Keep it concise (~200 lines is plenty for an initial spec).
4. Every Stack entry should reference an ADR if one exists.

## Heading anchors & sharding

### Stable level-2 heading anchors

- Each spec section is a `## ` level-2 heading and its slug is a stable anchor (e.g. `## API Surface` → `.spec/spec.md#api-surface`).
- Tickets and Sprint Plans point at these anchors (`.spec/spec.md#data-model`). **Do not rename or reorder** a heading without updating the inbound pointers — the anchor is a contract, not cosmetic.

### Cap-triggered domain shard (do NOT eager-shard)

- Keep the spec as a single file by default.
- ONLY when it grows past ~200 lines, shard a high-churn domain into `.spec/{domain}.md` and replace its section body with a one-line pointer to the shard.
- Sharding is reactive (triggered by the cap), never anticipatory. A small or new spec stays one file.

## Drift Detection

When patching an existing spec, compare:
1. **Directory pointer-map** vs actual directory listing. If new top-level dirs appeared (not in the diff), flag and investigate.
2. **Stack** section vs `package.json` dependencies. If deps were added/removed outside the diff, patch it.
3. **Architecture / Runtime view** vs actual component boundaries and flows. If a new service or package appeared, patch it.

If drift is detected: read the relevant new files, emit the corresponding delta hunks, and note in the commit message what drift was caught.

## Level of Detail

- **Types > prose.** Show the interface definition, not a paragraph describing it.
- **One level of nesting max** within a section (the enumerated subheads under Crosscutting Concepts & Patterns are the deliberate exception).
- **Target: ~200 lines.** If longer, the project grew significantly — shard a domain (see above) rather than padding the main file.
- **Reference, don't duplicate.** Point to ADRs for "why", to files for implementation, to `.sprint/backlog.md` for known debt/risk, and to `.brief/brief.md` for quality goals. The spec is a map, not the territory — it never restates them.

## What NOT to Include

- Implementation details (how a function works internally)
- Test descriptions or test file paths
- Build/CI/CD configuration
- Git workflow or branching strategy
- Environment variable values (just names)
- Comments in code or documentation standards
- **Changelogs, status columns, or verification matrices** — the spec describes what IS, not what changed or whether it passed. History is git; pass/fail lives in Issues and the cycle Goal.
- **A durable testable-requirements layer** (SHALL statements + scenarios) — this is DEFERRED; do NOT add it to the spec.
- **A steering file** — dropped from the model; do not create or reference one.
- **Verbatim API/schema dumps** — no pasted OpenAPI specs, full generated types, or schema dumps; show the shape and point to the source file (the spec is a map, not the territory).
