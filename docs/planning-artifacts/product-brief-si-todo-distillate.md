---
title: "Product Brief Distillate: SI-Todo"
type: llm-distillate
source: "product-brief-si-todo.md"
created: "2026-04-05"
purpose: "Token-efficient context for downstream PRD creation"
---

# SI-Todo — Detail Pack for PRD Creation

## Requirements Hints

- Cards must have: title, description, assignee (free text name), due date, priority (P1/P2/P3), tags/labels
- 6 columns as first-class board sections: Backlog, To Do, In Progress, Delegated, Blocked, Done — user confirmed these as columns, not card states
- Due-date aging thresholds: green (7+ days) → yellow (3–7 days) → red (<3 days) → pulsing red (overdue) — must be user-configurable
- Card colors are user-customizable to denote priority, work type, or assignee — applied as border/accent, while aging controls background — both signals coexist without conflict
- Light mode default with white or dotted grid background; dark mode available via toggle
- Data must persist to disk (file-based or SQLite) — localStorage alone is insufficient because VDI sessions can clear browser storage on logout
- Every change auto-saves — no manual save action required
- JSON export/import for backup and portability — in V1, not deferred
- Card archive/delete to prevent Done column from growing unbounded — in V1
- Filter/search by: assignee, priority, due date, tag
- Dashboard showing: total items, overdue count, items by assignee, items by priority

## Technical Context

- **Stack preference:** React or Next.js frontend, modern web stack, clean minimal UI
- **Deployment:** Localhost only — runs on user's laptop, accessed via browser at localhost
- **Storage:** Lightweight backend with file-based or SQLite persistence to disk — must survive server restarts, VDI shutdowns, browser closures
- **Auth:** None — single-user tool on localhost, no network exposure
- **Mobile:** Not required — desktop browser only
- **No responsive/mobile design work needed** — simplifies CSS and layout significantly

## Rejected Ideas and Explicit Non-Goals

- **No dark mode** → REVERSED: user initially said no dark mode, then corrected to toggleable light/dark mode
- **No multi-user or collaboration** — this is a personal tool, not a team platform. Intentionally "radically single-player"
- **No authentication/login** — single user on localhost, auth is unnecessary complexity
- **No sprint management, burndown charts, Gantt views** — this is not a project management suite
- **No third-party integrations for V1** — keep it self-contained
- **No cloud sync or multi-device sync for V1** — localhost only
- **No AI-powered features for V1**
- **No notifications or reminders for V1**
- **No mobile-specific design** — desktop browser only
- **Delegated and Blocked as card states/tags** → REJECTED: user explicitly chose full columns over card states

## Detailed User Scenarios

- User manages multiple engineering teams and workstreams simultaneously
- Typical board has ~30+ active items at any time
- Key workflow: assign task to team member → move card to Delegated with assignee name → follow up using filter ("show me everything delegated to Ahmed") → move to Done or back to In Progress
- VDI environment: user's laptop/desktop may be a virtual desktop that shuts down at end of day — data persistence is not optional, it's the core requirement
- User wants to open the board as first thing in the morning and immediately see what's overdue (red), at risk (yellow), and on track (green) without reading individual dates

## Competitive Intelligence

- **Trello** — dominant Kanban tool but had a widely panned redesign in mid-2025, driving active user exodus. Corporate-blocked for this user regardless.
- **Notion** — general workspace with Kanban views but overkill for a focused board; requires account and has learning curve
- **Lightweight OSS alternatives exist but are primitive:**
  - Nullboard: single HTML file, jQuery, no drag-and-drop polish
  - Kanbany: React/Next.js localStorage clone of Trello — closest competitor but lacks priority color-coding, delegation tracking, due-date aging, filtering, dashboards
  - Kanri: Tauri/Nuxt desktop-only app, not web-accessible
  - My Personal Kanban: basic, last meaningful update ~2013
- **Key gap in market:** No existing tool combines drag-and-drop Kanban + delegation tracking + priority color-coding + due-date aging + filtering/dashboard in one lightweight package
- **Market size:** Task management software market $5.71B (2025), to-do app sub-market $1.43B growing at 9.5% CAGR
- **Browser-based tools hold 77% market share** — validates web-app approach

## Open Questions for PRD Phase

- **Dashboard layout:** Separate view, sidebar, or overlay on the board? Brief says "at-a-glance" but doesn't specify placement
- **Aging threshold configurability:** Where does the user configure thresholds? Settings panel? Per-card override?
- **Card color rule system:** How complex? Simple manual per-card color picker vs. rule-based auto-coloring (e.g., "all P1 = red border") vs. both?
- **Backend technology choice:** File-based JSON vs. SQLite — needs decision during architecture
- **Drag-and-drop library:** dnd-kit is the leading React option — confirm during tech design
- **Board state on first launch:** Empty board with tutorial hints? Pre-populated sample cards?

## Scope Signals — Future / Maybe

- Export/import: **moved to V1** (was originally Vision)
- Card archive/delete: **moved to V1** (was originally not mentioned)
- 1:1 prep view (filter delegated by assignee): Vision
- Weekly digest export: Vision
- Recurring tasks: Vision
- Quick-capture via keyboard shortcut: Vision
- Optional cloud sync: Vision
- Archive and history (searchable completed items): Vision

## Review Panel Insights Worth Preserving

- **Skeptic:** localStorage is unreliable in VDI environments — confirmed by user, led to backend persistence requirement
- **Skeptic:** Done column will accumulate hundreds of cards within weeks without archive — led to adding archive/delete to V1
- **Skeptic:** Due-date aging is the headline feature but had zero specification — led to defining concrete thresholds
- **Opportunity:** Due-date aging is a behavioral design innovation, not just a feature — "the board tells you the truth even when you don't ask"
- **Opportunity:** The Delegated column is a category-defining feature that makes this a leadership tool, not a generic todo app
- **Opportunity:** Position against the real competitor: the neglected text file, not enterprise tools
- **Opportunity:** Future 1:1 prep view makes delegation tracking even more valuable
