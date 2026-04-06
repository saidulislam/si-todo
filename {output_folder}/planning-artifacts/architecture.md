---
stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8]
inputDocuments:
  - {output_folder}/planning-artifacts/prd.md
  - docs/planning-artifacts/product-brief-si-todo.md
  - docs/planning-artifacts/product-brief-si-todo-distillate.md
workflowType: 'architecture'
project_name: 'SI-Todo'
user_name: 'Saidul'
date: '2026-04-06'
lastStep: 8
status: 'complete'
completedAt: '2026-04-06'
---

# Architecture Decision Document

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

## Project Context Analysis

### Requirements Overview

**Functional Requirements:** 40 FRs across 8 areas — Board Structure (FR1–4), Card Management (FR5–10), Due-Date Aging (FR11–17), Card Customization (FR18–21), Filtering & Search (FR22–28), Dashboard (FR29–32), Persistence & Portability (FR33–37), Theming (FR38–40).

Architecturally, the FRs collapse to four core capabilities:
1. **Board state model + drag-and-drop** — six fixed columns, ordered cards, intra/inter-column moves
2. **Card CRUD with rich attributes** — title, description, assignee, due date, priority, tags, notes, custom accent color
3. **Real-time derived visuals** — aging color recalculation as time passes (client-side, no server tick)
4. **Local-first durable storage** — auto-save on every mutation, crash-safe, JSON export/import

**Non-Functional Requirements:** 14 NFRs split into Performance (NFR1–7) and Reliability/Data Integrity (NFR8–14).

The reliability NFRs are the most architecturally consequential:
- NFR8–10: Zero data loss across server restart, browser close, VDI shutdown → backend disk persistence is non-negotiable; localStorage alone is insufficient
- NFR11: Auto-save on every mutation → write path must be cheap and non-blocking
- NFR14: Resilient to unclean shutdown (process kill, power loss) → atomic write strategy required (write-temp + rename, or SQLite WAL)
- NFR12–13: Round-trip JSON export/import with no corruption → schema must be stable and serializable

Performance NFRs constrain the rendering layer:
- NFR2: <100ms drag response → drag-and-drop library must be performant; minimize re-renders during drag
- NFR6: Aging recalculation cannot stutter UI → recompute strategy must be O(n) over visible cards on a coarse timer (e.g., per minute) not per frame

**Scale & Complexity:**
- Primary domain: Full-stack web application (SPA frontend + local backend)
- Complexity level: Low-to-Medium — small functional surface, but durability and real-time visuals are non-trivial
- Estimated architectural components: ~6–8 (frontend SPA, state layer, drag-and-drop, aging engine, backend API, persistence layer, export/import, theming)
- Single user, single host — no concurrency, no multi-tenancy, no scaling concerns

### Technical Constraints & Dependencies

- **Localhost only** — no network exposure, no SSL/TLS, no CORS, no CDN, no auth
- **Chromium browsers only** (Chrome, Edge) — Firefox/Safari explicitly out of scope; simplifies CSS and API surface
- **Desktop only** — no responsive layouts, mouse + keyboard input model
- **Single developer, full-stack** — favors a unified language stack (e.g., TypeScript end-to-end) to minimize context switching
- **No deployment, no CI/CD, no IT involvement** — the app must start with a single command on the user's workstation
- **Drag-and-drop library** — explicitly deferred from PRD to architecture; dnd-kit named as candidate
- **Storage backend choice** — file-based JSON vs SQLite — explicitly deferred from PRD to architecture

### Cross-Cutting Concerns Identified

1. **Durability & crash-safety** — every mutation must reach disk atomically; affects backend write path, transaction model, and shutdown handling
2. **Auto-save coordination** — every UI mutation triggers a backend write; needs debouncing/batching strategy that still honors NFR11 (no lost mutations) and NFR5 (non-blocking)
3. **Real-time aging recalculation** — client-side periodic recompute affecting card backgrounds without re-rendering the entire board
4. **Filter/search state** — applies across the entire board view; must compose with drag-and-drop and not break card identity
5. **Theming (light/dark + grid background)** — CSS custom properties layered with aging backgrounds and custom accent borders without visual conflict (FR19)
6. **JSON schema stability** — export/import format doubles as the canonical data model; schema versioning may be needed for future-proofing
7. **Startup ergonomics** — single-command launch (backend + frontend) so the user opens the browser to a working board with no setup ritual

## Starter Template Evaluation

### Primary Technology Domain

Full-stack web application: React SPA (Vite) + Node/Fastify backend, single npm package, localhost-only.

### Starter Options Considered

| Option | Verdict |
|--------|---------|
| **Next.js** | Rejected — SSR, file-based routing, and middleware are unnecessary for a single-screen SPA on localhost. Adds conceptual surface area we never use. |
| **T3 Stack** (create-t3-app) | Rejected — bundles tRPC, NextAuth, Prisma. Auth is explicitly out of scope; Prisma is heavyweight against SQLite for ~5 tables; tRPC is overkill for a handful of CRUD endpoints. |
| **RedwoodJS / Blitz** | Rejected — opinionated full-stack frameworks designed for production deployment. Wrong shape for a localhost tool. |
| **Vite (react-ts) + sibling Fastify server** | **Selected** — minimal moving parts, single npm package, full TypeScript end-to-end, no unused machinery. |
| Monorepo (Turborepo/pnpm workspaces) | Rejected — single developer, single deployable; workspace tooling is pure overhead. |

### Selected Starter: Vite (react-ts) + Fastify in a single package

**Rationale for Selection:**
- **Fewest moving parts** — one `package.json`, one `node_modules`, one TypeScript config root with project references for the two sides.
- **No assumptions to delete** — Vite gives us React + TS + dev server + HMR and nothing else. Fastify gives us a fast, ergonomic HTTP layer and nothing else.
- **Single-command launch** — `concurrently` runs Vite dev server and Fastify together via one `npm run dev`. Vite proxies `/api/*` to Fastify so the SPA and API share an origin (no CORS).
- **TypeScript end-to-end** — shared types live in `src/shared/` and are imported by both client and server.
- **SQLite via `better-sqlite3`** — synchronous API (no Promise overhead for tiny local queries), battle-tested, supports WAL mode for crash safety. No ORM; hand-written SQL against a ~5-table schema is simpler than a query builder for this scale.

**Initialization Command:**

```bash
# Scaffold the frontend with Vite's React + TypeScript template
npm create vite@latest si-todo -- --template react-ts
cd si-todo

# Add backend + tooling
npm install fastify better-sqlite3
npm install -D @types/better-sqlite3 tsx concurrently
npm install -D tailwindcss @tailwindcss/vite

# (versions: use the latest at the time of init; pin in package.json)
```

**Architectural Decisions Provided by Starter:**

**Language & Runtime:**
- TypeScript end-to-end (strict mode)
- Node.js (LTS) for backend, modern Chromium for frontend
- ESM throughout

**Styling Solution:**
- Tailwind CSS v4 via `@tailwindcss/vite` plugin (zero-config, no `tailwind.config.js` needed for v4)
- CSS custom properties for theme tokens (light/dark mode toggle, dotted grid background)

**Build Tooling:**
- Vite for frontend dev server, HMR, and production build
- `tsx` for running the Fastify server in dev with TypeScript directly (no separate build step in dev)
- `tsc` for production server build (or keep `tsx` in production — it's a personal localhost tool)

**Testing Framework:**
- Vitest (ships natively with Vite, shares config, fastest setup) — added when first test is needed, not upfront
- No E2E framework in v1; manual smoke testing is sufficient for a single-user localhost tool

**Code Organization:**

```
si-todo/
├── src/
│   ├── client/          # React app (Vite root for frontend)
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── state/
│   │   └── main.tsx
│   ├── server/          # Fastify app
│   │   ├── routes/
│   │   ├── db/          # better-sqlite3 + migrations
│   │   └── index.ts
│   └── shared/          # Types shared by client + server
│       └── types.ts
├── data/                # SQLite database file lives here (gitignored)
├── index.html           # Vite entry
├── vite.config.ts       # Includes /api proxy → Fastify
├── tsconfig.json        # Project references for client/server/shared
└── package.json
```

**Development Experience:**
- `npm run dev` → `concurrently` starts Fastify (`tsx watch src/server/index.ts`) and Vite dev server. Vite proxies `/api/*` to Fastify on a fixed local port.
- HMR for the React side; tsx auto-restart for the server side
- Single `package.json`, single lockfile, no workspace tooling
- No Docker, no CI, no deployment pipeline — `npm run dev` is the entire ops story

**Note:** Project initialization using this command should be the first implementation story.

## Core Architectural Decisions

### Decision Priority Analysis

**Critical Decisions (Block Implementation):**
- SQLite via `better-sqlite3` with WAL journal mode
- Schema with hardcoded column enum + fractional positions for ordering
- Zod for shared client/server validation
- REST API with single-snapshot loading
- Zustand for client state management
- `@dnd-kit/core` + `@dnd-kit/sortable` for drag-and-drop
- Fastify bound to `127.0.0.1` only (no network exposure)

**Important Decisions (Shape Architecture):**
- 60s `setInterval` tick + `window` focus event for aging recalculation
- Per-card sequential mutation queue; 200ms debounce only on drag-move POSTs
- Tailwind for layout/state + CSS custom properties for theme tokens (dual-layer styling)
- Aging color → card background; custom accent color → card border (FR19 dual-signal)

**Deferred Decisions (Post-MVP):**
- SQLite file backup script (`npm run backup`)
- Whether to ship a production build or run `npm run dev` as the daily mode
- Schema versioning conventions beyond v1 (revisit when first migration is needed)

### Data Architecture

**Database:** SQLite via `better-sqlite3` (synchronous, native, no ORM).

**PRAGMAs at connection open:**
```sql
PRAGMA journal_mode=WAL;
PRAGMA synchronous=NORMAL;
PRAGMA foreign_keys=ON;
```
WAL gives crash-safety + concurrent reads. `synchronous=NORMAL` is the standard recommendation with WAL — safe against process crash; only loses data on OS crash with power loss, acceptable for a personal tool. Satisfies NFR8–NFR14.

**Schema (4 tables):**
- `cards` — `id` TEXT PK (uuid), `title`, `description`, `assignee`, `due_date` (ISO date or NULL), `priority` (P1/P2/P3 or NULL), `accent_color` (NULL = none), `column_id` (TEXT enum), `position` REAL (fractional), `created_at`, `updated_at`, `archived_at` NULL
- `tags` — `id` PK, `name` UNIQUE
- `card_tags` — `card_id`, `tag_id` (composite PK, ON DELETE CASCADE)
- `settings` — key/value singleton row (theme, aging thresholds, schema_version)

**No `columns` table.** The 6 columns are fixed (FR1) and live as a string enum in `src/shared/types.ts`. Removes a join, removes a migration, removes a foreign key.

**Ordering strategy:** fractional `position` (REAL). On drop, set new position to `(prev + next) / 2`. No sibling renumbering. Periodic rebalance only if positions get too small (effectively never at personal-board scale).

**Migrations:** hand-rolled. A `schema_version` row in `settings`; an array of migration SQL strings in `src/server/db/migrations.ts`; on startup, run any whose index > current version, in a transaction. No migration framework.

**Validation:** Zod schemas in `src/shared/schemas.ts`, used by both server (request body validation) and client (form validation). Types derived via `z.infer<>`. Single source of truth.

### Authentication & Security

**None.** No auth, sessions, CSRF tokens, or middleware. Fastify binds to `127.0.0.1` only (never `0.0.0.0`) to guarantee no network exposure even if the host firewall is permissive. This is the only security control.

### API & Communication Patterns

**Style:** plain REST/JSON over Fastify. ~10 endpoints total. tRPC/GraphQL rejected as machinery without benefit at this scale.

**Endpoints:**
```
GET    /api/state              → full board snapshot (cards + tags + settings)
POST   /api/cards              → create card
PATCH  /api/cards/:id          → partial update
DELETE /api/cards/:id          → delete
POST   /api/cards/:id/move     → { column_id, position } — explicit move endpoint
POST   /api/cards/:id/archive  → archive (sets archived_at)
GET    /api/tags
POST   /api/tags
PATCH  /api/settings           → partial settings update
GET    /api/export             → JSON export download
POST   /api/import             → JSON import (replaces full state)
```

**Single-snapshot loading:** `GET /api/state` returns everything on initial load. With ~50 cards this is ~10KB; no pagination, no caching. Client holds full board in memory; refetches on `window` focus to catch drift.

**Error handling:** Fastify default error handler; `{ error, code }` JSON for 4xx/5xx. Zod validation errors → 400 with field-level details.

**Not used:** CORS (same origin via Vite proxy), auth, rate limiting, OpenAPI docs.

### Frontend Architecture

**State management:** **Zustand**. Single store: `{ cards, tags, settings, filters, ui }`. Fine-grained selectors prevent over-rendering during drag (critical for NFR2 <100ms drag response). No TanStack Query — API surface is too small to justify the abstraction; cache is owned directly by the store.

**Drag-and-drop:** **`@dnd-kit/core` + `@dnd-kit/sortable`** (already named in PRD). Modern, accessible, performant, React-first.

**Routing:** **None.** Single-screen app. Modals (card editor, import confirm) are conditional renders, not routes.

**Aging recalculation:**
- A single `setInterval(..., 60_000)` bumps a `now` value in the Zustand store
- Card components compute their aging band as a memoized derived value from `(card.due_date, now)`
- Zustand fine-grained selectors re-render only the affected cards (satisfies NFR6 — no UI stutter)
- Also recompute on `window` `focus` event so the board is correct when returning from another tab

**Auto-save coordination:**
- Every store mutation immediately fires the corresponding API call (no debouncing in the general case — satisfies NFR11)
- Drag-move POSTs are debounced 200ms to coalesce rapid reorders
- Mutations queued sequentially per card to prevent out-of-order writes (small in-store mutation queue)
- On any failure: non-blocking toast + refetch `/api/state` to reconcile
- Optimistic updates throughout — UI reflects mutation immediately (satisfies NFR4 <200ms)

**Filtering & search:** entirely client-side. Full board is in memory; filters are derived selectors over the cards array. Zero server round-trips on filter changes (NFR3 <300ms is trivial).

**Styling architecture:**
- Tailwind for layout, spacing, typography, component states
- CSS custom properties (in `:root` and `[data-theme="dark"]`) for theme tokens (background, foreground, column bg, aging colors)
- Aging color applied as card `background` via a class derived from the aging band
- Custom accent color applied via inline `style={{ borderLeftColor: card.accent_color }}` — implements FR19 dual-signal: aging owns background, accent owns border
- Dark mode toggled by setting `data-theme` on `<html>`; preference persisted in `settings`

**Component structure:** flat. `<Board>` → `<Column>` (×6) → `<Card>`, plus `<CardEditorModal>`, `<FilterBar>`, `<Dashboard>`, `<Header>` (theme toggle, export/import). No design system, no Storybook.

### Infrastructure & Deployment

**None.** `npm run dev` is the entire ops story. Database file at `./data/si-todo.db` (gitignored). For "production" use, `npm run dev` is fine — it's a personal localhost tool.

**Optional production build path:** `npm run build` → Vite produces `dist/`, Fastify serves it as static files in production mode, same `/api/*` routes work. Decided per-story, not upfront.

**Logging:** Fastify's built-in pino logger at `info` level to stdout. No log files, no rotation, no observability stack.

**Backups:** JSON export is the primary backup story (FR36, NFR12). Optional `npm run backup` script that copies `data/si-todo.db` to `data/backups/si-todo-{timestamp}.db`. Deferred to a story.

### Decision Impact Analysis

**Implementation Sequence:**
1. Project init (Vite + Fastify scaffolding, single `package.json`, concurrently script)
2. SQLite + schema + migrations + PRAGMAs
3. Shared Zod schemas + TypeScript types
4. Fastify routes (`/api/state`, card CRUD, move, archive)
5. Zustand store + API client + mutation queue
6. Board UI shell (columns, cards, theming)
7. dnd-kit integration (intra/inter-column moves)
8. Card editor modal (create/edit forms with Zod validation)
9. Aging color engine + tick + window focus reload
10. Filtering, search, dashboard
11. Export/import
12. Theme toggle + dark mode polish

**Cross-Component Dependencies:**
- Shared Zod schemas are upstream of every other concern — must land first after scaffolding
- Aging recalculation depends on the Zustand store shape and selector pattern being established
- Drag-and-drop depends on the fractional position scheme being implemented in both schema and move endpoint
- Optimistic mutation queue is the spine the entire UI hangs from — get it right early

## Implementation Patterns & Consistency Rules

### Naming Patterns

**Database:**
- Tables: `snake_case`, plural (`cards`, `tags`, `card_tags`, `settings`)
- Columns: `snake_case` (`due_date`, `created_at`, `column_id`)
- Primary key: always `id`
- Foreign key: `<table_singular>_id` (`card_id`, `tag_id`)
- Timestamps: `created_at`, `updated_at`, `archived_at` — all stored as ISO 8601 strings (TEXT), not unix epochs

**API:**
- Path: `/api/<resource-plural>` in `kebab-case` if multi-word
- Resource paths use plural (`/api/cards`)
- Sub-actions on a resource use a verb segment: `/api/cards/:id/move`, `/api/cards/:id/archive`
- Path params: `:id` style (Fastify convention)
- Query params: `camelCase` (`includeArchived=true`)
- HTTP methods: `GET` (read), `POST` (create + actions), `PATCH` (partial update), `DELETE` (delete)
- Status codes: `200` success, `201` created, `204` no content, `400` validation, `404` not found, `500` server error

**TypeScript / React:**
- React component files: `PascalCase.tsx` (`CardEditorModal.tsx`)
- Hook files: `camelCase.ts` starting with `use` (`useBoardStore.ts`)
- Utility/server/schema files: `camelCase.ts` (`agingColors.ts`, `cardRoutes.ts`)
- Components: `PascalCase`
- Hooks: `useCamelCase`
- Functions/variables: `camelCase`
- Constants: `SCREAMING_SNAKE_CASE` only for true module-level constants (`COLUMN_IDS`, `AGING_THRESHOLDS_DEFAULT`)
- Types/interfaces: `PascalCase`. Prefer `type` aliases over `interface` unless declaration merging is needed
- Zod schemas: `<Name>Schema` (`CardSchema`, `CardCreateSchema`); inferred type `<Name>` (`type Card = z.infer<typeof CardSchema>`)

### Structure Patterns

**Co-location over centralization.** Tests live next to source as `<file>.test.ts`. No global `__tests__/` folder.

**Server organization (`src/server/`):**
- `index.ts` — Fastify bootstrap, plugin registration, listen on `127.0.0.1`
- `db/` — `connection.ts` (better-sqlite3 + PRAGMAs), `migrations.ts`, `migrations/*.sql`
- `routes/` — one file per resource: `cardRoutes.ts`, `tagRoutes.ts`, `settingsRoutes.ts`, `stateRoutes.ts`, `importExportRoutes.ts`
- `repositories/` — one file per table: `cardRepository.ts`, `tagRepository.ts`, `settingsRepository.ts`. **All SQL lives here.** Routes call repository functions; routes do not write SQL.
- No service layer — at this scale, route handlers calling repositories directly is fewer indirections

**Client organization (`src/client/`):**
- `main.tsx` — React entry
- `App.tsx` — top-level shell
- `components/` — flat folder, one file per component (`Board.tsx`, `Column.tsx`, `Card.tsx`, `CardEditorModal.tsx`, `FilterBar.tsx`, `Dashboard.tsx`, `Header.tsx`, `Toast.tsx`)
- `state/` — `boardStore.ts` (Zustand store + actions), `mutationQueue.ts`
- `api/` — `client.ts` (typed fetch wrapper, one function per endpoint)
- `hooks/` — custom hooks (`useAgingTick.ts`, `useFilteredCards.ts`)
- `lib/` — pure utility functions (`agingColors.ts`, `position.ts` for fractional ordering math)
- `styles/` — `index.css` (Tailwind directives + CSS custom properties for theme tokens)

**Shared (`src/shared/`):**
- `types.ts` — column enum, card types, settings types (mostly re-exports from schemas)
- `schemas.ts` — Zod schemas (single source of truth for shapes; both server and client import from here)
- `constants.ts` — `COLUMN_IDS`, `AGING_THRESHOLDS_DEFAULT`, etc.

### Format Patterns

**API responses:**
- Success: return the resource directly, no wrapper. `GET /api/cards/:id` → `{ id, title, ... }`. `GET /api/state` → `{ cards: [...], tags: [...], settings: {...} }`
- Error: `{ error: string, code: string, details?: unknown }` with appropriate HTTP status. Zod errors put field issues in `details`.
- No envelope (`{ data, error }`) — direct responses are simpler and TypeScript-friendly with Zod parsing on the client

**JSON field naming:** `camelCase` end-to-end. Repositories convert `snake_case` DB columns ↔ `camelCase` JS objects at the boundary. **No `snake_case` ever leaves the repository layer.**

**Dates:** ISO 8601 strings everywhere (DB, API, client state). The client converts to `Date` only at the moment of comparison/display. Never serialize `Date` objects directly — always `.toISOString()`.

**Booleans:** `true`/`false` in JSON, `INTEGER` 0/1 in SQLite, converted at the repository boundary.

**IDs:** `crypto.randomUUID()` generated server-side on create. Always strings.

### Communication Patterns

**State management (Zustand):**
- One store: `useBoardStore`. No store splitting.
- Actions are methods on the store, named as verbs: `createCard`, `updateCard`, `deleteCard`, `moveCard`, `setFilter`, `clearFilters`
- Every mutating action: (1) optimistically update store, (2) enqueue API call, (3) on failure show toast + refetch `/api/state`
- Selectors live in component files via `useBoardStore(state => ...)` or in `hooks/` for reusable derived state
- Never mutate store state directly outside of `set()` callbacks

**API client:**
- `src/client/api/client.ts` exports one async function per endpoint, each returning the parsed response
- All responses validated against the same Zod schemas the server uses (defense in depth + perfect type inference)
- Errors thrown as `ApiError` with `{ message, code, status }`

### Process Patterns

**Error handling:**
- Server: throw inside route handlers; a Fastify `setErrorHandler` converts thrown errors to the standard `{ error, code }` shape. Zod validation errors auto-converted to 400. Unknown errors logged at `error` level and returned as 500 with generic message.
- Client: `try/catch` around every API call in store actions. On failure: `toast.error(message)` + refetch `/api/state` to reconcile optimistic state.
- No error boundaries needed for v1 — single-screen app, optimistic + reconcile is sufficient.

**Loading states:**
- Initial board load: a single `isLoading` flag in the store, true until first `/api/state` resolves. Show a minimal "Loading…" placeholder.
- Per-mutation loading: none. Optimistic updates make the UI feel instant; the toast on failure is the only feedback for failed writes.
- No spinners on individual cards or columns.

**Validation timing:**
- Client: Zod-parse form input on submit (not on keystroke), show field errors inline
- Server: Zod-parse request body in every route handler before touching the repository. The canonical schema lives in `src/shared/schemas.ts`

**Logging:**
- Server: Fastify's pino at `info` level. Log every mutation (level `info`) and every error (level `error`). No request logging in dev (too noisy).
- Client: `console.error` for caught failures. No client logging infrastructure.

### Enforcement Guidelines

**All AI agents (and humans) MUST:**
- Import shapes from `src/shared/schemas.ts` — never re-declare a Card/Tag/Settings type elsewhere
- Put all SQL in `src/server/repositories/` — never inline SQL in route handlers
- Convert `snake_case` ↔ `camelCase` at the repository boundary, never anywhere else
- Use ISO 8601 strings for dates in DB, API, and store; convert to `Date` only at use site
- Add new mutating actions to the store, not directly in components — components call store actions
- Bind Fastify to `127.0.0.1`, never `0.0.0.0`
- Keep components flat under `src/client/components/` (no nested feature folders) until there's a clear reason otherwise
- Co-locate tests as `<file>.test.ts`

**Anti-patterns to avoid:**
- ❌ Direct `fetch()` calls from components — go through `src/client/api/client.ts`
- ❌ Adding a `services/` or `controllers/` layer "for separation" — routes call repositories, that's it
- ❌ Splitting Zustand store into multiple stores — one store, scoped selectors
- ❌ Introducing TanStack Query, RTK, Redux, MobX, Jotai, Recoil — Zustand is the decision
- ❌ Introducing Prisma, Drizzle, Kysely, or any ORM/query builder — hand-written SQL in repositories
- ❌ Adding routing (React Router, TanStack Router) — modals, not routes
- ❌ snake_case fields in API or store
- ❌ Storing dates as numbers or `Date` objects in the DB or store
- ❌ Nested component folders (`components/board/cards/Card.tsx`) — flat is fine for ~10 components

## Project Structure & Boundaries

### Complete Project Directory Structure

```
si-todo/
├── README.md
├── package.json                      # Single package, all deps
├── package-lock.json
├── tsconfig.json                     # Root: project references → client/server/shared
├── tsconfig.client.json              # JSX, DOM lib, Vite types
├── tsconfig.server.json              # Node types, no DOM
├── tsconfig.shared.json              # Pure TS, no DOM/Node lib
├── vite.config.ts                    # React plugin, /api proxy → 127.0.0.1:3001
├── .gitignore                        # node_modules, dist, data/, .env
├── .env.example                      # PORT=3001 (example only)
├── index.html                        # Vite SPA entry
│
├── data/                             # SQLite database lives here (gitignored)
│   └── .gitkeep
│
├── src/
│   ├── shared/                       # Used by BOTH client and server
│   │   ├── schemas.ts                # Zod schemas (single source of truth)
│   │   ├── types.ts                  # Re-exports from schemas + ColumnId enum
│   │   └── constants.ts              # COLUMN_IDS, AGING_THRESHOLDS_DEFAULT
│   │
│   ├── server/                       # Fastify backend
│   │   ├── index.ts                  # Bootstrap: connect db, register routes, listen on 127.0.0.1
│   │   ├── env.ts                    # PORT, DB_PATH (with sane defaults)
│   │   ├── errorHandler.ts           # Fastify setErrorHandler → {error,code,details}
│   │   ├── db/
│   │   │   ├── connection.ts         # better-sqlite3 + PRAGMAs (WAL, synchronous=NORMAL, FK)
│   │   │   ├── migrations.ts         # Runner: applies migrations in transaction by index
│   │   │   └── migrations/
│   │   │       └── 001_initial.sql   # cards, tags, card_tags, settings tables
│   │   ├── repositories/             # ALL SQL lives here
│   │   │   ├── cardRepository.ts     # CRUD, move, archive, list, snake↔camel mapping
│   │   │   ├── cardRepository.test.ts
│   │   │   ├── tagRepository.ts
│   │   │   └── settingsRepository.ts
│   │   └── routes/                   # One file per resource; routes call repositories
│   │       ├── stateRoutes.ts        # GET /api/state
│   │       ├── cardRoutes.ts         # POST/PATCH/DELETE /api/cards, move, archive
│   │       ├── tagRoutes.ts          # GET/POST /api/tags
│   │       ├── settingsRoutes.ts     # PATCH /api/settings
│   │       └── importExportRoutes.ts # GET /api/export, POST /api/import
│   │
│   └── client/                       # React SPA
│       ├── main.tsx                  # React root + initial /api/state fetch
│       ├── App.tsx                   # Top-level shell: <Header/> <FilterBar/> <Dashboard/> <Board/> <Toast/>
│       ├── styles/
│       │   └── index.css             # Tailwind directives + :root/[data-theme=dark] CSS vars
│       ├── api/
│       │   └── client.ts             # Typed fetch wrapper, one fn per endpoint, Zod-parsed responses
│       ├── state/
│       │   ├── boardStore.ts         # Zustand store: cards, tags, settings, filters, ui, actions
│       │   └── mutationQueue.ts      # Per-card sequential queue + drag-move debounce
│       ├── hooks/
│       │   ├── useAgingTick.ts       # 60s setInterval + window focus → bumps `now`
│       │   └── useFilteredCards.ts   # Derived selector applying filters/search
│       ├── lib/
│       │   ├── agingColors.ts        # (dueDate, now, thresholds) → 'green'|'yellow'|'red'|'overdue'|'none'
│       │   ├── agingColors.test.ts
│       │   ├── position.ts           # Fractional ordering math (between, rebalance)
│       │   └── position.test.ts
│       └── components/               # Flat folder
│           ├── Board.tsx             # 6-column layout, dnd-kit DndContext
│           ├── Column.tsx            # SortableContext per column
│           ├── Card.tsx              # Card visual: aging bg + accent border + priority/tags
│           ├── CardEditorModal.tsx   # Create/edit form, Zod validation on submit
│           ├── FilterBar.tsx         # Assignee/priority/due/tag/search controls
│           ├── Dashboard.tsx         # Totals, overdue, by-assignee, by-priority
│           ├── Header.tsx            # Theme toggle, export/import buttons, "+ Add card"
│           └── Toast.tsx             # Non-blocking error notifications
```

### Architectural Boundaries

**API boundary (client ↔ server):** the only contract between client and server is the `/api/*` HTTP surface. Both sides validate against the same Zod schemas in `src/shared/schemas.ts`. Vite dev server proxies `/api/*` to Fastify on `127.0.0.1:3001`, giving SPA and API a single origin (no CORS).

**Repository boundary (server-internal):** all SQL is confined to `src/server/repositories/`. Routes are forbidden from importing `better-sqlite3` directly. Repositories own the `snake_case` ↔ `camelCase` translation; everything outside this layer sees only `camelCase` JS objects.

**Store boundary (client-internal):** all server state lives in the Zustand `boardStore`. Components are forbidden from calling `src/client/api/client.ts` directly — they call store actions. The store owns optimistic updates, the mutation queue, and reconciliation on failure.

**Pure-function boundary:** `src/client/lib/` contains only pure functions (`agingColors`, `position`). They take inputs, return outputs, no side effects, no React, no store access. This is where the most testable logic concentrates.

**Shared boundary:** `src/shared/` is import-only for both sides. It must not import from `client/` or `server/`. It must not depend on `react`, `fastify`, `better-sqlite3`, or any DOM/Node API. Enforced by `tsconfig.shared.json` lib settings.

### Requirements to Structure Mapping

| FR Group | Lives In |
|---|---|
| FR1–4: Board layout & drag-and-drop | `Board.tsx`, `Column.tsx`, `Card.tsx`, `lib/position.ts`, `cardRoutes.ts` (move endpoint), `cardRepository.move()` |
| FR5–10: Card CRUD | `CardEditorModal.tsx`, `boardStore.createCard/updateCard/deleteCard/archiveCard`, `cardRoutes.ts`, `cardRepository.ts` |
| FR11–17: Due-date aging | `lib/agingColors.ts`, `hooks/useAgingTick.ts`, `Card.tsx`, `settingsRepository.ts` (thresholds), `styles/index.css` (color vars) |
| FR18–21: Card customization (accent + priority + tags) | `Card.tsx`, `CardEditorModal.tsx`, `tagRepository.ts`, `tagRoutes.ts` |
| FR22–28: Filtering & search | `FilterBar.tsx`, `hooks/useFilteredCards.ts`, `boardStore.filters` |
| FR29–32: Dashboard | `Dashboard.tsx` (pure derived selectors over store) |
| FR33–35: Persistence + auto-save | `db/connection.ts` (WAL), `repositories/*`, `state/mutationQueue.ts`, `state/boardStore.ts` |
| FR36–37: JSON export/import | `importExportRoutes.ts`, `Header.tsx` |
| FR38–40: Theming | `Header.tsx` (toggle), `styles/index.css`, `settingsRepository.ts` (persistence) |

| NFR | Enforcement Location |
|---|---|
| NFR1 (load <2s) | Single `/api/state` fetch; no waterfall |
| NFR2 (drag <100ms) | dnd-kit + Zustand fine-grained selectors in `Card.tsx` |
| NFR3 (filter <300ms) | Client-side filtering in `useFilteredCards.ts` |
| NFR4 (CRUD <200ms) | Optimistic updates in `boardStore` actions |
| NFR5 (auto-save non-blocking) | `mutationQueue.ts` (async, off the render path) |
| NFR6 (aging no stutter) | 60s tick + memoized selectors |
| NFR8–10 (zero data loss) | `db/connection.ts` PRAGMAs (WAL), Fastify writes via better-sqlite3 |
| NFR11 (auto-save every mutation) | Every store action enqueues an API call |
| NFR12–13 (export/import roundtrip) | Shared Zod schemas validate both export and import |
| NFR14 (crash-safe) | SQLite WAL + atomic transactions in repository methods |

### Integration Points

**Internal communication (client → server):** HTTP only, via `src/client/api/client.ts`. No WebSocket, no SSE, no long-poll. Client refetches `/api/state` on window focus to catch drift.

**Internal communication (component → store → API):**
```
Component → store action → (1) optimistic state update
                         → (2) mutationQueue.enqueue(apiCall)
                         → (3) on failure: toast + refetch /api/state
```

**External integrations:** **none**. No third-party services, no CDNs, no telemetry, no analytics.

**Data flow:**
1. **Startup:** Fastify boots → opens SQLite (WAL on) → runs pending migrations → listens on `127.0.0.1:3001`. Vite dev server starts → opens `localhost:5173` → React mounts → calls `GET /api/state` → store hydrates → board renders.
2. **Mutation:** User action → component calls store action → optimistic update → mutationQueue → API call → repository → SQLite write (WAL, atomic). On success: nothing more. On failure: toast + `GET /api/state` to reconcile.
3. **Aging tick:** 60s `setInterval` → store sets `now` → memoized selectors recompute aging band per card → React re-renders only changed cards.

### File Organization Patterns

**Configuration:** All config files at repo root (`tsconfig*.json`, `vite.config.ts`, `package.json`). No nested config sprawl.

**Source:** Strict three-way split (`client/`, `server/`, `shared/`) under `src/`. The split is enforced by tsconfig project references and lib settings — `shared/` cannot accidentally import a DOM or Node API.

**Tests:** Co-located with source as `<file>.test.ts`. Vitest auto-discovers. No `tests/` or `__tests__/` folders.

**Assets:** None. No images, fonts, or static files needed for v1 (Tailwind handles all visuals; emoji or SVG inline if icons are needed). If assets become necessary, they go in Vite's `public/` folder.

**Data:** SQLite database file at `data/si-todo.db`. The `data/` folder is gitignored except for `.gitkeep`. Database backups (if implemented) go to `data/backups/`.

### Development Workflow Integration

**Dev mode:** `npm run dev` runs `concurrently -k -n server,client "tsx watch src/server/index.ts" "vite"`.
- Server hot-restart via `tsx watch` on any file change in `src/server/` or `src/shared/`
- Client HMR via Vite on any change in `src/client/` or `src/shared/`
- Both processes share `src/shared/`; both pick up shared changes automatically

**Build:**
- `npm run build` → `tsc -b && vite build` → emits `dist/` with the SPA + compiled server
- `npm run start` → `node dist/server/index.js` (server serves the SPA from `dist/client/` as static files)
- Production mode is optional; `npm run dev` is the daily-driver mode for a personal tool

**Deployment:** None. The project lives on the developer's workstation.

## Architecture Validation Results

### Coherence Validation ✅

**Decision Compatibility:** All technology choices compose cleanly.
- TypeScript end-to-end + Vite + React + Fastify + better-sqlite3 + Zod + Zustand + dnd-kit + Tailwind v4 are all current, actively maintained, and routinely combined in production codebases.
- No version conflicts (single `package.json`, single lockfile).
- No competing concerns: no two libraries try to own state, routing, data fetching, or styling.

**Pattern Consistency:** Patterns reinforce decisions.
- `camelCase` everywhere outside the repository layer matches the Zod schema convention and removes a class of mapping bugs.
- "All SQL in repositories" matches "no ORM" — both decisions push toward the same simple, transparent data layer.
- "One Zustand store" matches "no React Router" matches "no TanStack Query" — a single, coherent reduction in client-side complexity.

**Structure Alignment:** The directory tree directly reflects the boundaries.
- `shared/` enforced by tsconfig lib settings prevents accidental cross-boundary imports.
- `repositories/` directory makes the SQL boundary visible at the file system level.
- Flat `components/` discourages premature feature-folder hierarchy.

### Requirements Coverage Validation ✅

**Functional Requirements (40/40 covered):** Every FR maps to a specific file or component in the §"Requirements to Structure Mapping" table. Spot checks:
- FR1–4 (board + drag-drop): `Board.tsx` + `Column.tsx` + `Card.tsx` + dnd-kit + `position.ts` + `cardRepository.move`. ✅
- FR11–17 (aging system): `agingColors.ts` (pure) + `useAgingTick.ts` (60s + focus) + threshold persistence in `settings`. ✅
- FR16 (configurable thresholds): handled via `settings` table + `PATCH /api/settings`. ✅
- FR19 (dual-signal coexistence): explicitly designed via background-vs-border separation. ✅
- FR33–37 (persistence + JSON export/import): backend-first storage, auto-save on every mutation, dedicated import/export routes, schemas validated by shared Zod. ✅

**Non-Functional Requirements (14/14 covered):** Every NFR maps to an enforcement location in the §"Requirements to Structure Mapping" table.
- Performance NFRs (1–7) covered by optimistic updates, fine-grained selectors, client-side filtering, single-snapshot loading, and memoized aging recompute.
- Reliability NFRs (8–14) covered by SQLite WAL, atomic transactions, mutation queue, and shared Zod validation on the export/import roundtrip.

### Implementation Readiness Validation ✅

**Decision Completeness:** All critical decisions are documented with rationale. Deferred decisions are explicitly marked and don't block implementation.

**Structure Completeness:** The project tree names every file an AI agent will need to create, with a one-line purpose for each. New work should always have an obvious home.

**Pattern Completeness:** Naming, structure, format, communication, and process patterns are all specified, with explicit anti-patterns enumerated to head off the most likely AI agent drifts (alternate state libs, ORMs, routing, snake_case leakage, nested component folders).

### Gap Analysis Results

**Critical Gaps:** None.

**Important Gaps (worth noting, not blocking):**
- **No accessibility test strategy.** PRD calls keyboard nav and contrast "basic," not WCAG. Acceptable for v1; revisit if scope grows.
- **No backup automation.** JSON export is the manual safety net. A `npm run backup` script is on the deferred list — implement when first needed.
- **Schema versioning convention beyond v1 is undefined.** First migration after v1 is the right time to establish it; premature design now would just be guessed.

**Minor Gaps (nice-to-have refinements):**
- A README explaining `npm run dev` and the data folder location should be the second-to-last story.
- Optional: a one-shot script to seed the database with sample cards for demoing/testing.

### Validation Issues Addressed

No critical or important issues required architectural changes during validation. The architecture is internally consistent and externally complete against the PRD.

### Architecture Completeness Checklist

**✅ Requirements Analysis**
- [x] Project context thoroughly analyzed
- [x] Scale and complexity assessed (Low-to-Medium)
- [x] Technical constraints identified (Node-only, localhost, Chromium, single user)
- [x] Cross-cutting concerns mapped (durability, auto-save coordination, aging recompute, theming)

**✅ Architectural Decisions**
- [x] Critical decisions documented with rationale
- [x] Technology stack fully specified (TS, React, Vite, Fastify, better-sqlite3, Zod, Zustand, dnd-kit, Tailwind v4, Vitest)
- [x] Integration patterns defined (REST, optimistic + reconcile, mutation queue)
- [x] Performance considerations addressed (optimistic UI, fine-grained selectors, client-side filtering)

**✅ Implementation Patterns**
- [x] Naming conventions established (DB snake_case, JS camelCase, components PascalCase)
- [x] Structure patterns defined (co-located tests, flat components, repository SQL boundary)
- [x] Communication patterns specified (component → store → mutationQueue → API → repository)
- [x] Process patterns documented (error handling, loading states, validation timing, logging)

**✅ Project Structure**
- [x] Complete directory structure defined
- [x] Component boundaries established (API, repository, store, pure-function, shared)
- [x] Integration points mapped (HTTP only, no external integrations)
- [x] Requirements to structure mapping complete (FR table + NFR table)

### Architecture Readiness Assessment

**Overall Status:** READY FOR IMPLEMENTATION

**Confidence Level:** High. The architecture is small, opinionated, internally consistent, and traceable to every PRD requirement. The "fewest moving parts" constraint has been honored throughout.

**Key Strengths:**
- Minimal surface area: one package, one store, one DB, one HTTP layer, one styling system
- Hard boundaries enforced by both file system layout and tsconfig lib settings
- Crash-safe persistence via battle-tested SQLite WAL
- Optimistic UI + mutation queue gives instant feel without sacrificing durability
- Shared Zod schemas provide single source of truth for shapes across client, server, and import/export
- Zero external dependencies at runtime (no cloud, no telemetry, no auth providers)

**Areas for Future Enhancement:**
- WCAG-formal accessibility pass if scope expands
- Schema versioning convention (defer until first real migration)
- Automated backup script (defer to first story that needs it)
- Production build path is wired but not exercised — adopt it if `npm run dev` ever feels heavyweight as the daily driver

### Implementation Handoff

**AI Agent Guidelines:**
- Follow all architectural decisions exactly as documented
- Use implementation patterns consistently across all components — especially the camelCase boundary, the SQL-only-in-repositories rule, and the components-don't-call-API-directly rule
- Respect project structure and boundaries — don't add `services/`, `controllers/`, nested feature folders, or alternate state libraries
- Refer to this document for all architectural questions; if a question isn't answered here, ask before guessing

**First Implementation Priority:**
Project initialization story — scaffold per the §"Selected Starter" command:

```bash
npm create vite@latest si-todo -- --template react-ts
cd si-todo
npm install fastify better-sqlite3 zod zustand @dnd-kit/core @dnd-kit/sortable
npm install -D @types/better-sqlite3 tsx concurrently tailwindcss @tailwindcss/vite vitest
```

Then restructure into the `src/{shared,server,client}/` layout, set up tsconfig project references, and add the `npm run dev` concurrently script. Subsequent stories follow the implementation sequence in §"Decision Impact Analysis".
