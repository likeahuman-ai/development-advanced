# Stack Detection Reference

Detect the project's tech stack from `package.json` dependencies and config files. Report as a concise string (e.g., "Next.js + Tailwind CSS + Prisma + PostgreSQL").

## Framework Detection

| Dependency | Stack label |
|---|---|
| `next` | Next.js |
| `react` (without next) | React |
| `vue` | Vue |
| `svelte` / `@sveltejs/kit` | Svelte / SvelteKit |
| `nuxt` | Nuxt |
| `express` | Express |
| `fastify` | Fastify |
| `hono` | Hono |
| `astro` | Astro |
| `remix` / `@remix-run/*` | Remix |
| `angular` / `@angular/core` | Angular |

## Styling Detection

| Dependency / Config | Stack label |
|---|---|
| `tailwindcss` or `tailwind.config.*` exists | Tailwind CSS |
| `styled-components` | Styled Components |
| `@emotion/react` | Emotion |
| `sass` / `*.scss` files | Sass |
| `vanilla-extract` | Vanilla Extract |

## Backend / Database Detection

| Dependency / Config | Stack label |
|---|---|
| `convex` | Convex |
| `prisma` / `@prisma/client` | Prisma |
| `drizzle-orm` | Drizzle |
| `mongoose` | MongoDB (Mongoose) |
| `@supabase/supabase-js` | Supabase |
| `firebase` / `firebase-admin` | Firebase |
| `pg` / `postgres` | PostgreSQL |
| `better-sqlite3` / `sql.js` | SQLite |
| `typeorm` | TypeORM |
| `sequelize` | Sequelize |

## Auth Detection

| Dependency | Stack label |
|---|---|
| `@clerk/nextjs` / `@clerk/*` | Clerk |
| `next-auth` / `@auth/core` | Auth.js |
| `passport` | Passport |
| `lucia` | Lucia Auth |

## Testing Detection

| Dependency | Stack label |
|---|---|
| `vitest` | Vitest |
| `jest` | Jest |
| `@testing-library/*` | Testing Library |
| `playwright` / `@playwright/test` | Playwright |
| `cypress` | Cypress |

## Component Libraries

| Dependency | Stack label |
|---|---|
| `@radix-ui/*` | Radix |
| `@headlessui/*` | Headless UI |
| `@shadcn/ui` or `components.json` exists | shadcn/ui |
| `@chakra-ui/*` | Chakra UI |
| `@mui/material` | Material UI |
| `storybook` / `@storybook/*` | Storybook |

## Usage

Read `package.json` → match dependencies against this table → concatenate detected labels with " + ".

Only include what's actually present. Don't guess. If nothing matches a category, skip it.
