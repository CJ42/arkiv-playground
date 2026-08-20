# AGENTS.md

Guidance for coding agents working in this monorepo. This file is the single source of truth for agent guidance — `CLAUDE.md` and `.github/copilot-instructions.md` are thin pointers to it.

## Scope and Priorities

1. Follow this file for repo-specific workflows and conventions.
2. Follow direct user instructions over this file when they conflict.
3. Keep changes minimal, targeted, and consistent with existing code.
   
## Skills (task-specific workflow guides)

Reusable process checklists live in `.claude/skills/*/SKILL.md` (plain Markdown — usable by any tool; Claude Code loads them automatically). Consult the matching one BEFORE starting these task types:

| Skill | When to use |
|---|---|
| [`verify-changes`](./.claude/skills/verify-changes/SKILL.md) | After any edit — minimal check set per touched area |
| [`ui-conventions`](./.claude/skills/ui-conventions/SKILL.md) | Any UI change — i18n, dark mode, three-state views |

## Environment and Toolchain

- Bun as package manager.
- Node.js: project docs target Node 22.
- TypeScript is strict.

## Repository structure

```
arkiv-app/
├── public/
├── src/
│   ├── app/                        # routing only: layouts, pages, route handlers
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── globals.css
│   │   ├── error.tsx
│   │   ├── loading.tsx
│   │   ├── not-found.tsx
│   │   ├── entities/
│   │   │   ├── page.tsx
│   │   │   ├── _components/        # private: used by this route only
│   │   │   └── [id]/page.tsx
│   │   └── api/
│   │       └── health/route.ts
│   ├── components/
│   │   ├── ui/                     # shadcn/ui primitives, domain-agnostic
│   │   └── layout/                 # navbar, footer, page shells
│   │   └── shared/                 # custom created component, shared and re-usable 
│   ├── features/                   # domain UI, vertical slices, one folder per domain area
│   │   └── entity-explorer/
│   │       ├── components/
│   │       ├── hooks/
│   │       ├── lib/
│   │       ├── types.ts
│   │       └── index.ts            # the ONLY public entry point
│   ├── lib/                        # cross-cutting infra, cn, formatters, query keys
│   │   ├── arkiv/                  # SDK client, query builders
│   │   ├── wagmi.ts
│   │   └── utils.ts                # cn(), formatters
│   ├── config/                     # env parsing, chains, feature flags, wagmi, viem, arkiv clients
│   ├── hooks/                      # genuinely app-wide hooks only
│   ├── services/                   # throws user-friendly errors for React Query
│   ├── types/                      # ambient + genuinely shared types
│   ├── styles/                     # only truly global styles
│   └── proxy.ts                    # Next 16's renamed middleware
├── next.config.ts
├── tsconfig.json
└── package.json
```

## Code Style Guidelines

### General TypeScript Rules

- Prefer explicit types on exported APIs and `import type` for type-only imports.
- Keep strict-null and unknown-safe patterns; narrow with guards.
- Avoid `any`; if unavoidable, keep scope tiny and document briefly.
- Prefix intentionally unused variables/parameters with `_` (e.g., `_unused`).
- Prefer small pure functions for transformations and parsing logic.

### ESLint Key Rules (collector)

- `no-console`: **`console.log` is forbidden** — use `console.info`, `console.warn`, or `console.error` instead.
- `@typescript-eslint/no-unused-vars`: warn-level; variables/args prefixed with `_` are exempted.
- `@typescript-eslint/no-explicit-any`: warn-level in source code.
- **Test files** (`*.test.ts`, `__tests__/**`): `no-explicit-any` and `no-console` are turned off.
- Nothing ever imports from `app/`, and no feature imports another feature's internals — only its `index.ts`. Enforced with `no-restricted-imports` patterns in `eslint.config.mjs` to ensure this convention survives.

### Imports and Module Boundaries

- Keep import groups logically ordered:
  1) platform/native modules,
  2) third-party packages,
  3) workspace/internal modules.
- In web app code, prefer alias imports via `@/*` for internal modules.

### Formatting and Layout

- Match local file style instead of forcing global formatting changes.

### Naming Conventions

- Filenames: kebab-case (e.g., `auth.service.ts`, `use-gateway.ts`).
- React components: PascalCase exports; file names remain kebab-case.
- React hooks: `useXxx` naming.
- Types/interfaces/enums: PascalCase.
- Variables/functions: camelCase.
- Environment variables/constants: `UPPER_SNAKE_CASE`.
- Database-origin fields may use snake_case (`is_active`, `created_at`); preserve API contracts.

### Error Handling and Logging

- In `catch`, use `error: unknown`, then narrow via `instanceof Error`.
- Return structured API errors for HTTP handlers (e.g., `{ error: string }`) with proper status codes.
- Add contextual logs for operational failures (network, DB, background jobs).
- Use timeouts for external network calls where appropriate (`AbortSignal.timeout(...)`).
- Prefer resilient flows (`Promise.allSettled`) when partial results are acceptable.

### Frontend Patterns

- Respect Next.js App Router conventions (`app/[locale]/...`).
- Add `"use client"` only for client components/hooks that need it.
- Prefer React Query hooks for server state and caching.
- Reuse shared utility helpers (`cn`, formatting utilities) instead of ad-hoc duplicates.
- **UI components**: shadcn/ui (`new-york` style) + Tailwind CSS v4 + `lucide-react` icons.
  - Config: `app/components.json`.
  - Base UI primitives live in `app/components/ui/`. They MUST know nothing about the app domain.
  - components/layout
- Everything in `app/features/` MUST know everything about the app domain.
- Everything in `app/layout/` is the shell.

Following this convention for creating new components:
- Folder-per-component with colocated styles
- Nested `components/` for sub-parts

_Example_:

```
├── src/
  ├── components/
    ├── ui/
      ├── Card/
        ├── Card.module.scss
        ├── Card.tsx
        ├── index.ts
```

Where `index.ts` contains:

```ts title="index.ts"
export { default } from './NavBar';
```

- `config/` separate from `constants/`. `config/wagmi.ts` (wiring, may read env) vs `constants/*.ts` (frozen domain values).

