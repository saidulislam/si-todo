---
stepsCompleted: [1, 2, 3, 4]
status: 'complete'
inputDocuments:
  - {output_folder}/planning-artifacts/prd.md
  - {output_folder}/planning-artifacts/architecture.md
---

# SI-Todo - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for SI-Todo, decomposing the requirements from the PRD and Architecture into implementable stories.

## Requirements Inventory

### Functional Requirements

**Board Structure & Layout**
- FR1: User can view a Kanban board with six fixed columns: Backlog, To Do, In Progress, Delegated, Blocked, Done
- FR2: User can view all cards within each column in a vertically scrollable list
- FR3: User can reorder cards within a column via drag-and-drop
- FR4: User can move cards between columns via drag-and-drop

**Card Management**
- FR5: User can create a new card with title, description, assignee, due date, priority (P1/P2/P3), and tags
- FR6: User can edit any field on an existing card
- FR7: User can delete a card permanently
- FR8: User can archive a card from the Done column
- FR9: User can assign a card directly to a specific column during creation
- FR10: User can add free-text notes to a card

**Due-Date Aging System**
- FR11: System automatically displays cards with a green background when the due date is 7+ days away
- FR12: System automatically displays cards with a yellow background when the due date is 3–7 days away
- FR13: System automatically displays cards with a red background when the due date is fewer than 3 days away
- FR14: System automatically displays cards with a pulsing red background when the due date has passed
- FR15: Cards with no due date display with a neutral background (no aging color)
- FR16: User can configure the aging threshold values (days for green, yellow, red transitions)
- FR17: System recalculates aging colors in real time while the board is open

**Card Customization & Visual System**
- FR18: User can assign a custom accent color to any card
- FR19: Custom accent colors display as card border/accent, coexisting with aging background colors without conflict
- FR20: Cards display their priority level (P1/P2/P3) visually
- FR21: Cards display their assigned tags visually

**Filtering & Search**
- FR22: User can filter the board by assignee
- FR23: User can filter the board by priority level
- FR24: User can filter the board by due date range
- FR25: User can filter the board by tag
- FR26: User can combine multiple filters simultaneously
- FR27: User can clear all active filters to return to the full board view
- FR28: User can search cards by text content (title, description)

**Dashboard & Summary**
- FR29: User can view a summary dashboard showing total active card count
- FR30: User can view a count of overdue cards in the dashboard
- FR31: User can view a breakdown of cards by assignee in the dashboard
- FR32: User can view a breakdown of cards by priority level in the dashboard

**Data Persistence & Portability**
- FR33: System automatically saves all changes to backend storage without manual save action
- FR34: System persists all board data across server restarts
- FR35: System persists all board data across browser closures
- FR36: User can export the complete board state as a JSON file
- FR37: User can import a previously exported JSON file to restore board state

**Appearance & Theming**
- FR38: User can toggle between light mode and dark mode
- FR39: Light mode displays a dotted grid background option
- FR40: User's theme preference persists across sessions

### NonFunctional Requirements

**Performance**
- NFR1: Board initial load completes in under 2 seconds with up to 50 active cards
- NFR2: Drag-and-drop card movement responds in under 100ms (feels instant)
- NFR3: Filter and search results display in under 300ms
- NFR4: Card creation, editing, and deletion reflect on the board in under 200ms
- NFR5: Auto-save completes in the background without blocking user interaction
- NFR6: Due-date aging color recalculation does not cause visible UI stutter or delay
- NFR7: Dashboard summary updates without perceptible delay after board changes

**Reliability & Data Integrity**
- NFR8: Zero data loss across server restarts — all board state is recoverable
- NFR9: Zero data loss across browser closures — no pending changes are lost
- NFR10: Zero data loss across VDI session shutdowns — backend persists to disk, not memory
- NFR11: Auto-save triggers on every user mutation — no manual save action required
- NFR12: JSON export produces a complete, valid, re-importable snapshot of all board data
- NFR13: JSON import restores board state fully without data corruption or loss
- NFR14: Backend storage is resilient to unclean shutdown (process kill, power loss) — data corruption must not occur from incomplete writes

### Additional Requirements

**From Architecture:**
- **Starter scaffold (Epic 1, Story 1):** Initialize project via `npm create vite@latest si-todo -- --template react-ts` and add Fastify, better-sqlite3, Zod, Zustand, @dnd-kit, Tailwind v4, tsx, concurrently, vitest. Restructure into `src/{shared,server,client}/` layout with tsconfig project references. Add `npm run dev` running Vite + `tsx watch` Fastify under `concurrently`.
- **TypeScript project references:** Three tsconfigs (client/server/shared) with strict mode and lib boundaries enforced.
- **SQLite + WAL:** `better-sqlite3` connection module sets `journal_mode=WAL`, `synchronous=NORMAL`, `foreign_keys=ON` at open.
- **Schema migrations runner:** Hand-rolled migration loop with `schema_version` tracking; first migration creates `cards`, `tags`, `card_tags`, `settings`.
- **Shared Zod schemas:** Single source of truth in `src/shared/schemas.ts`; types derived via `z.infer<>`.
- **Fractional ordering math:** Pure utility in `src/client/lib/position.ts` for between(prev, next) ordering.
- **Repository pattern:** All SQL confined to `src/server/repositories/`; routes call repositories.
- **camelCase API boundary:** Repositories convert snake_case ↔ camelCase; nothing outside the repository sees snake_case.
- **Fastify bound to 127.0.0.1 only.**
- **Optimistic mutation queue:** Per-card sequential queue in `src/client/state/mutationQueue.ts`; 200ms debounce on drag-move POSTs only.
- **Aging tick:** 60s setInterval + window focus event bumps `now` in store.
- **Vite proxy `/api/*` → Fastify** for same-origin development.
- **Vitest co-located tests** as `<file>.test.ts`.
- **No auth, no CORS, no SSL, no Docker, no CI** — explicit non-requirements.

### UX Design Requirements

_No standalone UX Design document was produced. UX requirements are folded into the FRs (board layout, dual-signal visual system, theming, dashboard) and captured in the Architecture's styling section (Tailwind v4 + CSS custom properties, aging-as-background + accent-as-border)._

### FR Coverage Map

| FR | Epic | Notes |
|---|---|---|
| FR1 | Epic 1 | Six-column board layout |
| FR2 | Epic 1 | Vertically scrollable card lists |
| FR3 | Epic 1 | Drag-reorder within column |
| FR4 | Epic 1 | Drag-move between columns |
| FR5 | Epic 1 | Card creation with full attributes |
| FR6 | Epic 1 | Edit any field on a card |
| FR7 | Epic 1 | Permanent delete |
| FR8 | Epic 2 | Archive card from Done |
| FR9 | Epic 1 | Create card directly into a column |
| FR10 | Epic 2 | Free-text notes on a card |
| FR11 | Epic 1 | Aging green (7+ days) |
| FR12 | Epic 1 | Aging yellow (3–7 days) |
| FR13 | Epic 1 | Aging red (<3 days) |
| FR14 | Epic 1 | Aging pulsing red (overdue) |
| FR15 | Epic 1 | Neutral background when no due date |
| FR16 | Epic 2 | Configurable aging thresholds |
| FR17 | Epic 1 | Real-time aging recalculation |
| FR18 | Epic 2 | Custom accent color per card |
| FR19 | Epic 2 | Accent + aging coexist (border vs background) |
| FR20 | Epic 1 | Priority displayed visually |
| FR21 | Epic 2 | Tags displayed visually |
| FR22 | Epic 3 | Filter by assignee |
| FR23 | Epic 3 | Filter by priority |
| FR24 | Epic 3 | Filter by due date range |
| FR25 | Epic 3 | Filter by tag |
| FR26 | Epic 3 | Combine multiple filters |
| FR27 | Epic 3 | Clear all filters |
| FR28 | Epic 3 | Text search (title, description) |
| FR29 | Epic 3 | Dashboard total count |
| FR30 | Epic 3 | Dashboard overdue count |
| FR31 | Epic 3 | Dashboard by assignee |
| FR32 | Epic 3 | Dashboard by priority |
| FR33 | Epic 1 | Auto-save on every mutation |
| FR34 | Epic 1 | Persistence across server restarts |
| FR35 | Epic 1 | Persistence across browser closures |
| FR36 | Epic 4 | JSON export |
| FR37 | Epic 4 | JSON import |
| FR38 | Epic 5 | Light/dark mode toggle |
| FR39 | Epic 5 | Dotted grid background option |
| FR40 | Epic 5 | Theme preference persistence |

## Epic List

### Epic 1: Working Board with Persistent Cards
The fallback MVP. After this epic, Saidul can open the board, create cards, drag them between six columns, see aging colors, and trust that nothing is lost on restart.
**FRs covered:** FR1, FR2, FR3, FR4, FR5, FR6, FR7, FR9, FR11, FR12, FR13, FR14, FR15, FR17, FR20, FR33, FR34, FR35
**NFR coverage:** NFR1–NFR11, NFR14 (the entire performance + crash-safe persistence story is established here)

### Epic 2: Delegation & Card Detail
Turns the board into a leadership tool. Adds the things that make a card carry context: tags, free-text notes, archive, accent colors, configurable thresholds.
**FRs covered:** FR8, FR10, FR16, FR18, FR19, FR21

### Epic 3: Filtering, Search & Dashboard
Turns "see all cards" into "see exactly the slice I need." All client-side derived selectors over the in-memory store.
**FRs covered:** FR22, FR23, FR24, FR25, FR26, FR27, FR28, FR29, FR30, FR31, FR32

### Epic 4: Data Portability
The user's safety net. JSON export and import for backup, migration, and pre-triage snapshots.
**FRs covered:** FR36, FR37
**NFR coverage:** NFR12, NFR13

### Epic 5: Theming & Visual Polish
Light/dark mode, dotted grid background, persisted preference.
**FRs covered:** FR38, FR39, FR40

## Epic 1: Working Board with Persistent Cards

After this epic, Saidul has a usable board with all six columns, cards he can create/edit/delete/drag, aging colors, and crash-safe persistence. This is the fallback MVP and the gate before everything else.

### Story 1.1: Project Scaffold and Dev Environment

As the developer,
I want a single-package TypeScript project with Vite + React + Fastify wired together,
So that I can run the entire app with one `npm run dev` command and start building features.

**Acceptance Criteria:**

**Given** a clean working directory
**When** I run the project init commands from the architecture document and add the `concurrently` script
**Then** `npm run dev` starts both the Vite dev server and Fastify backend together
**And** Vite serves the React app on `localhost:5173` with HMR working
**And** Fastify listens on `127.0.0.1:3001` (never `0.0.0.0`)
**And** Vite proxies `/api/*` requests to Fastify so the SPA and API share an origin
**And** the project tree matches the `src/{shared,server,client}/` layout from the architecture document
**And** TypeScript project references for client/server/shared are configured with strict mode and the `shared/` tsconfig forbids DOM and Node lib imports
**And** Tailwind v4 is installed via `@tailwindcss/vite` and a hello-world page renders with a Tailwind class applied
**And** `data/` exists and is gitignored except for `.gitkeep`
**And** the README documents `npm run dev` as the only command needed

### Story 1.2: SQLite Connection and Initial Schema Migration

As the developer,
I want a SQLite database with a crash-safe connection and the initial schema in place,
So that the backend has durable storage for cards from day one.

**Acceptance Criteria:**

**Given** the project scaffold from Story 1.1
**When** the Fastify server starts
**Then** `src/server/db/connection.ts` opens `data/si-todo.db` via `better-sqlite3`
**And** the connection sets `PRAGMA journal_mode=WAL`, `PRAGMA synchronous=NORMAL`, and `PRAGMA foreign_keys=ON`
**And** `src/server/db/migrations.ts` runs any pending migrations inside a transaction, tracked by a `schema_version` row in `settings`
**And** migration `001_initial.sql` creates the `cards` table with columns matching the architecture (id TEXT PK, title, description, assignee, due_date, priority, accent_color, column_id, position REAL, created_at, updated_at, archived_at) and the `settings` table with the `schema_version` row seeded to `1`
**And** killing the Fastify process mid-write does not corrupt the database (verifiable by running migrations again on the same file with no errors)
**And** the connection module is unit-tested for PRAGMA application

### Story 1.3: Shared Zod Schemas and Type Boundaries

As the developer,
I want shared Zod schemas for cards and settings used by both client and server,
So that there is a single source of truth for data shapes and types are inferred everywhere.

**Acceptance Criteria:**

**Given** the project scaffold and database schema in place
**When** I create `src/shared/schemas.ts` and `src/shared/types.ts` and `src/shared/constants.ts`
**Then** `schemas.ts` exports `CardSchema`, `CardCreateSchema` (no id/timestamps), and `CardUpdateSchema` (all fields optional) using Zod
**And** `types.ts` re-exports inferred types via `z.infer<>` (`Card`, `CardCreate`, `CardUpdate`)
**And** `constants.ts` exports `COLUMN_IDS` as a const tuple `['backlog','todo','in-progress','delegated','blocked','done']` and a `ColumnId` union type
**And** `CardSchema.column_id` validates against `COLUMN_IDS`
**And** dates are typed as ISO 8601 strings, not `Date` objects
**And** the `shared/` folder imports nothing from `client/` or `server/` and uses no DOM or Node APIs (enforced by tsconfig lib settings)

### Story 1.4: Card Repository with CRUD Operations

As the developer,
I want a card repository encapsulating all SQL with snake_case ↔ camelCase mapping,
So that route handlers never write SQL and never see snake_case fields.

**Acceptance Criteria:**

**Given** the database connection and shared schemas
**When** I create `src/server/repositories/cardRepository.ts`
**Then** the repository exports `listAll()`, `findById(id)`, `create(input)`, `update(id, patch)`, `delete(id)`, `move(id, columnId, position)`, and `archive(id)` functions
**And** all SQL lives only in this file — no other server file imports `better-sqlite3`
**And** every function returns or accepts `camelCase` JS objects matching the shared `Card` type
**And** snake_case ↔ camelCase translation happens only inside this module
**And** `create` generates the id via `crypto.randomUUID()` and sets `created_at`/`updated_at` to ISO strings
**And** `update` automatically bumps `updated_at` and only modifies the columns present in the patch
**And** `move` updates `column_id` and `position` atomically in a single statement
**And** `archive` sets `archived_at` to the current ISO timestamp
**And** the repository has co-located unit tests (`cardRepository.test.ts`) that exercise every function against an in-memory or temp-file SQLite database

### Story 1.5: Fastify Server with State and Card Routes

As the developer,
I want Fastify routes for loading the full board state and performing card CRUD,
So that the client can hydrate from the server and persist mutations.

**Acceptance Criteria:**

**Given** the card repository from Story 1.4
**When** I create `src/server/routes/stateRoutes.ts`, `src/server/routes/cardRoutes.ts`, and register them in `src/server/index.ts`
**Then** `GET /api/state` returns `{ cards: Card[], tags: [], settings: { agingThresholds, schemaVersion } }` as a single snapshot
**And** `POST /api/cards` validates the body with `CardCreateSchema`, calls `cardRepository.create`, and returns the new card with status 201
**And** `PATCH /api/cards/:id` validates the body with `CardUpdateSchema`, calls `cardRepository.update`, and returns the updated card
**And** `DELETE /api/cards/:id` calls `cardRepository.delete` and returns 204
**And** `POST /api/cards/:id/move` accepts `{ columnId, position }`, validates, calls `cardRepository.move`, and returns the updated card
**And** route files do not import `better-sqlite3` directly — only the repository
**And** a Fastify `setErrorHandler` returns errors as `{ error, code, details? }` with appropriate HTTP status codes
**And** Zod validation errors return 400 with field-level details
**And** Fastify binds to `127.0.0.1` only

### Story 1.6: Client API Client, Zustand Store, and Mutation Queue

As the developer,
I want a typed API client and Zustand store with an optimistic mutation queue,
So that components can call store actions and the UI feels instant.

**Acceptance Criteria:**

**Given** the Fastify routes from Story 1.5
**When** I create `src/client/api/client.ts`, `src/client/state/boardStore.ts`, and `src/client/state/mutationQueue.ts`
**Then** `api/client.ts` exports one async function per endpoint (`fetchState`, `createCard`, `updateCard`, `deleteCard`, `moveCard`) that returns Zod-parsed responses
**And** API errors throw an `ApiError` with `{ message, code, status }`
**And** `boardStore.ts` exports a Zustand store with `{ cards, tags, settings, filters, ui: { isLoading, now }, ...actions }`
**And** the store exposes actions: `loadState`, `createCard`, `updateCard`, `deleteCard`, `moveCard`
**And** every mutating action (1) optimistically updates store state, (2) enqueues the API call via `mutationQueue.enqueue(cardId, fn)`, (3) on failure shows a toast and refetches `/api/state` to reconcile
**And** `mutationQueue.ts` runs mutations sequentially per `cardId` to prevent out-of-order writes
**And** drag-move POSTs are debounced 200ms inside the queue
**And** `loadState` is called once on app mount and sets `isLoading: false` on success
**And** components are forbidden from calling `api/client.ts` directly — they call store actions

### Story 1.7: Board Layout with Six Columns

As Saidul,
I want to see the board with all six columns rendered,
So that I have the visual frame in place even before any cards exist.

**Acceptance Criteria:**

**Given** the store and hydration from Story 1.6
**When** I open `localhost:5173`
**Then** the board renders with six columns in this order: Backlog, To Do, In Progress, Delegated, Blocked, Done
**And** each column shows its title at the top
**And** each column displays the cards belonging to it, ordered by `position` ascending, in a vertically scrollable list
**And** an empty column shows a subtle "No cards" placeholder
**And** while initial state is loading, a minimal "Loading…" placeholder is shown
**And** the layout uses Tailwind for spacing and a CSS custom property for column background color (so it can be themed later)
**And** `Board.tsx`, `Column.tsx`, and `Card.tsx` exist as separate files in `src/client/components/`

### Story 1.8: Card Creation Modal

As Saidul,
I want to create a new card with title, description, assignee, due date, priority, and target column,
So that I can capture work the moment it appears.

**Acceptance Criteria:**

**Given** the board renders
**When** I click "+ Add card" in the header
**Then** a modal opens with input fields for title (required), description, assignee, due date, priority (P1/P2/P3 select with "none" option), and target column (defaults to To Do)
**And** the form validates input on submit using `CardCreateSchema` and shows field-level errors inline
**And** clicking "Save" creates the card via `useBoardStore().createCard`, the card appears immediately at the top of the chosen column (optimistic), and the modal closes
**And** clicking "Cancel" or pressing Escape closes the modal without creating a card
**And** if the API call fails, a toast appears, the optimistic card is removed, and the store refetches state
**And** card creation reflects on the board in under 200ms (NFR4)

### Story 1.9: Card Editing and Deletion

As Saidul,
I want to edit any field on an existing card or delete it permanently,
So that I can keep cards accurate and remove ones that no longer matter.

**Acceptance Criteria:**

**Given** at least one card exists on the board
**When** I click a card
**Then** the same modal from Story 1.8 opens, pre-filled with the card's current values
**And** clicking "Save" updates the card via `useBoardStore().updateCard` (optimistic) and the changes appear immediately
**And** clicking "Delete" prompts a confirmation, then permanently removes the card via `useBoardStore().deleteCard`
**And** deleting an optimistic card removes it from the board immediately and the API call runs in the background
**And** failures roll back via toast + refetch
**And** the store reuses the same `CardUpdateSchema` for client-side validation
**And** edit and delete reflect on the board in under 200ms (NFR4)

### Story 1.10: Drag-and-Drop Within and Between Columns

As Saidul,
I want to drag cards within and between columns to reorder and reclassify them,
So that I can rearrange my board fluidly without typing.

**Acceptance Criteria:**

**Given** multiple cards exist across columns
**When** I drag a card to a new position within the same column
**Then** the card visually moves immediately (optimistic) and `boardStore.moveCard` is called with the new fractional position computed via `lib/position.ts`
**And** when I drag a card to a different column, it appears in the new column at the dropped position immediately
**And** the move POST to `/api/cards/:id/move` is debounced 200ms in the mutation queue so rapid successive moves coalesce
**And** the fractional position math (`between(prev, next)`) lives in `src/client/lib/position.ts` as a pure function with co-located tests
**And** drag-and-drop uses `@dnd-kit/core` and `@dnd-kit/sortable` with one `DndContext` in `Board.tsx` and one `SortableContext` per column
**And** drag response time is under 100ms (NFR2)
**And** dragging is possible across all six columns including into/out of Delegated and Blocked

### Story 1.11: Due-Date Aging Color System

As Saidul,
I want cards to automatically display aging background colors based on their due date,
So that I can see what's overdue, at risk, and on track without reading any dates.

**Acceptance Criteria:**

**Given** cards exist with various due dates (some 7+ days out, some 3–7 days, some <3 days, some overdue, some with no due date)
**When** the board renders
**Then** cards with due dates 7+ days away display a green background
**And** cards with due dates 3–7 days away display a yellow background
**And** cards with due dates fewer than 3 days away display a red background
**And** cards with due dates in the past display a pulsing red background (CSS keyframe animation)
**And** cards with no due date display with a neutral background
**And** the aging band is computed by a pure function `agingColors.ts` that takes `(dueDate, now, thresholds)` and returns `'green' | 'yellow' | 'red' | 'overdue' | 'none'`
**And** `agingColors.test.ts` covers all five bands and the boundary days
**And** aging colors are applied via CSS custom properties so they can be re-themed in Epic 5
**And** the `useAgingTick` hook in `src/client/hooks/` runs a 60-second `setInterval` that bumps a `now` value in the store
**And** the hook also recomputes on `window` `focus` event
**And** card components use a memoized selector so only the cards whose band changed re-render — the entire board does not re-render every minute
**And** there is no visible UI stutter on re-render (NFR6)
**And** the priority level (P1/P2/P3) is displayed visually on each card (FR20)

## Epic 2: Delegation & Card Detail

After this epic, cards carry full leadership context: tags, free-text notes, custom accent colors, archive workflow, and configurable aging thresholds.

### Story 2.1: Tag Repository, Routes, and Store Integration

As the developer,
I want tags as a first-class entity with repository, routes, and store support,
So that cards can carry tags and the UI can list/create them.

**Acceptance Criteria:**

**Given** the migrations system from Story 1.2
**When** I add migration `002_tags.sql`
**Then** the migration creates `tags` (id INTEGER PK, name TEXT UNIQUE) and `card_tags` (card_id, tag_id, composite PK, ON DELETE CASCADE) tables
**And** `src/server/repositories/tagRepository.ts` exposes `listAll()`, `create(name)`, `setCardTags(cardId, tagIds[])`, `getCardTagIds(cardId)`
**And** `src/server/routes/tagRoutes.ts` exposes `GET /api/tags` and `POST /api/tags` (validated by a `TagSchema` in `shared/schemas.ts`)
**And** `GET /api/state` returns `tags` populated from the new repository
**And** the `Card` shape in shared schemas now includes `tagIds: number[]` and the card repository populates/persists this field via the join table
**And** the boardStore exposes `tags`, an `addTag(name)` action, and updates `createCard`/`updateCard` to accept `tagIds`

### Story 2.2: Tag Display and Selection on Cards

As Saidul,
I want to assign tags to a card and see them displayed on the card,
So that I can categorize work by topic (e.g. `infrastructure`, `P1`, `1:1-prep`).

**Acceptance Criteria:**

**Given** the tag backend from Story 2.1
**When** I open the card editor modal
**Then** a tag input lets me type to filter existing tags or create a new one (typing "infrastructure" + Enter creates the tag if it doesn't exist)
**And** selected tags display as removable pill chips in the modal
**And** saving the card persists the selected `tagIds` via `updateCard`
**When** I view a card on the board
**Then** the card displays its tags as small visual chips below the title
**And** cards with no tags show no tag area

### Story 2.3: Free-Text Notes Field on Cards

As Saidul,
I want to add and edit free-text notes on a card,
So that I can capture context that doesn't fit in the description.

**Acceptance Criteria:**

**Given** the card editor modal
**When** I look at the modal layout
**Then** a "Notes" textarea appears below the description, supports multi-line input, and persists via `updateCard`
**And** notes are stored in a new `notes TEXT` column added by migration `003_card_notes.sql`
**And** the shared `CardSchema` is updated to include `notes` as an optional string
**And** cards on the board do not display the notes inline (notes are detail-only)

### Story 2.4: Card Accent Color Picker

As Saidul,
I want to assign a custom accent color to a card,
So that I can visually distinguish important or related cards beyond aging colors.

**Acceptance Criteria:**

**Given** the card editor modal
**When** I look at the modal layout
**Then** an accent color picker offers a small palette of preset colors plus a "none" option
**And** selecting a color persists `accentColor` via `updateCard`
**When** I view a card on the board
**Then** the accent color is rendered as a left border on the card via inline `style={{ borderLeftColor }}`
**And** the accent border coexists with the aging background — both are visible simultaneously, neither overrides the other (FR19)
**And** cards with no accent color show no left border

### Story 2.5: Archive Card from Done Column

As Saidul,
I want to archive cards from the Done column,
So that completed work disappears from view but isn't lost forever.

**Acceptance Criteria:**

**Given** a card sits in the Done column
**When** I open the card editor modal
**Then** an "Archive" button appears (only for cards in Done)
**And** clicking Archive calls `boardStore.archiveCard` which calls `POST /api/cards/:id/archive`, sets `archived_at`, and the card disappears from the board immediately
**And** the `cardRepository.listAll()` query filters out archived cards by default
**And** archived cards are still in the database (not deleted) so they can be restored later or appear in JSON exports
**And** archive reflects on the board in under 200ms

### Story 2.6: Configurable Aging Threshold Settings

As Saidul,
I want to configure the day thresholds for green/yellow/red aging colors,
So that I can tune the heat map to my own sense of urgency.

**Acceptance Criteria:**

**Given** the settings table exists
**When** I open a settings panel (accessible from the header)
**Then** three numeric inputs let me set the green-threshold (default 7), yellow-threshold (default 3), and changes are validated (green > yellow ≥ 1)
**And** saving calls `PATCH /api/settings` which persists the new thresholds in the `settings` table
**And** the boardStore reloads `settings.agingThresholds` and the aging color recompute uses the new values immediately
**And** thresholds are read by `agingColors.ts` from the store rather than hardcoded
**And** the settings persist across server restarts

## Epic 3: Filtering, Search & Dashboard

After this epic, Saidul can slice the board by any combination of assignee, priority, due range, tag, and free-text search, and see live summary counts.

### Story 3.1: Filter Bar with Assignee, Priority, Due Date, and Tag Filters

As Saidul,
I want a filter bar with controls for assignee, priority, due date range, and tag,
So that I can narrow the board to exactly the cards I need to focus on.

**Acceptance Criteria:**

**Given** cards exist with varied assignees, priorities, due dates, and tags
**When** the board renders
**Then** a filter bar appears above the columns with: an assignee dropdown (populated from existing card assignees), a priority dropdown (P1/P2/P3/Any), a due date range selector (overdue / next 3 days / next 7 days / any), and a tag dropdown (populated from `tags`)
**And** selecting any filter immediately narrows the visible cards across all columns
**And** filters compose — selecting assignee=Fatima AND priority=P1 shows only Fatima's P1 cards
**And** a "Clear filters" button resets all filters in one click
**And** filter state lives in `boardStore.filters` and is applied via the `useFilteredCards` selector
**And** filtering is entirely client-side over the in-memory cards array
**And** filter results display in under 300ms (NFR3)
**And** active filters are visually indicated (e.g., highlighted control or pill row)

### Story 3.2: Text Search Across Title and Description

As Saidul,
I want to search cards by text content in title or description,
So that I can quickly find a specific card without scanning columns.

**Acceptance Criteria:**

**Given** cards exist with varied titles and descriptions
**When** I type in the search input in the filter bar
**Then** cards whose title or description contains the search text (case-insensitive) remain visible and others are hidden
**And** search composes with the other filters from Story 3.1 (search + assignee + priority all apply together)
**And** clearing the search input restores the non-search-filtered view
**And** the "Clear filters" button also clears the search input
**And** search updates in under 300ms

### Story 3.3: Summary Dashboard

As Saidul,
I want a summary dashboard showing total cards, overdue count, by-assignee breakdown, and by-priority breakdown,
So that I can see the state of the board at a glance during morning check.

**Acceptance Criteria:**

**Given** cards exist on the board
**When** I look at the dashboard area (a compact panel near the header)
**Then** it displays the total count of active (non-archived) cards
**And** it displays the count of overdue cards (due date in the past)
**And** it displays a breakdown of cards by assignee (e.g. "Ahmed: 3, Fatima: 2, Unassigned: 5")
**And** it displays a breakdown of cards by priority (e.g. "P1: 4, P2: 7, P3: 2, None: 3")
**And** all counts are derived live from the store via memoized selectors — no separate API call
**And** counts update without perceptible delay after any card mutation (NFR7)
**And** when a filter is active, the dashboard reflects the filtered view (so filtered-by-Fatima dashboard shows only Fatima's totals)

## Epic 4: Data Portability

After this epic, Saidul can export the entire board to a JSON file and re-import it. The export is the canonical backup format.

### Story 4.1: JSON Export

As Saidul,
I want to export the complete board state to a JSON file,
So that I have a portable backup I can restore from later.

**Acceptance Criteria:**

**Given** the board has cards, tags, and settings
**When** I click "Export" in the header
**Then** the browser downloads `si-todo-export-{ISO-date}.json`
**And** the file contains `{ schemaVersion, exportedAt, cards: [...], tags: [...], settings: {...} }` (including archived cards)
**And** the export is generated by `GET /api/export` on the server, which queries all repositories
**And** the export shape is defined by an `ExportSchema` Zod schema in `shared/schemas.ts` so the same schema validates both export output and import input
**And** re-importing a fresh export produces an identical board (NFR12 round-trip)
**And** the export contains every field necessary to reconstruct the board (no data loss)

### Story 4.2: JSON Import with Confirmation

As Saidul,
I want to import a previously exported JSON file to restore board state,
So that I can recover from data loss or migrate between machines.

**Acceptance Criteria:**

**Given** I have a valid SI-Todo export JSON file
**When** I click "Import" in the header and select the file
**Then** a confirmation modal appears warning that import will REPLACE the current board state
**And** confirming sends the file contents to `POST /api/import`
**And** the server validates the payload with `ExportSchema` (rejects with 400 + details if invalid)
**And** on success, the server replaces `cards`, `tags`, and `settings` in a single transaction
**And** the client refetches `/api/state` and the board re-renders with the imported data
**And** an invalid JSON file shows a clear error toast and does not modify the board
**And** the round-trip (export → wipe → import) reconstructs the board exactly (NFR13)

## Epic 5: Theming & Visual Polish

After this epic, Saidul can switch between light and dark mode, optionally enable a dotted grid background in light mode, and the choice persists across sessions.

### Story 5.1: Light/Dark Mode Toggle with Persistence

As Saidul,
I want to toggle between light and dark mode with my choice persisted,
So that the board matches my environment without me re-setting it every session.

**Acceptance Criteria:**

**Given** the board renders with default light theme
**When** I click the theme toggle in the header
**Then** the entire board immediately switches to dark mode
**And** the theme is implemented via `data-theme="dark"` on `<html>` and CSS custom properties in `styles/index.css` for both `:root` and `[data-theme="dark"]`
**And** all aging colors (green/yellow/red/overdue) have appropriate dark-mode variants with sufficient contrast
**And** all column backgrounds, card backgrounds, and text colors use the theme variables (no hard-coded colors)
**And** the chosen theme is persisted by calling `PATCH /api/settings` with `{ theme: 'light' | 'dark' }`
**And** the `settings` table is updated via migration `004_theme_setting.sql` to include a `theme` key
**And** on next load, `GET /api/state` returns the theme and the client applies it before the first paint (no flash)

### Story 5.2: Dotted Grid Background Option

As Saidul,
I want an optional dotted grid background in light mode,
So that the board has a subtle texture that makes it feel like a workspace, not a blank screen.

**Acceptance Criteria:**

**Given** the board is in light mode
**When** I open the settings panel and toggle "Dotted grid background" on
**Then** a subtle dotted grid appears as the page background (CSS `background-image` with radial-gradient dots)
**And** toggling it off removes the grid
**And** the grid is only available in light mode (in dark mode the toggle is hidden or disabled)
**And** the choice persists via `PATCH /api/settings` with `{ dottedGrid: boolean }`
**And** the `settings` table is updated via migration `005_dotted_grid_setting.sql`
**And** the grid is purely cosmetic — it does not interfere with drag-and-drop or card readability
