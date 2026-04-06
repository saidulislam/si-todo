# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

> **This section is auto-populated.** After `/bmad-product-brief` completes, update this section with the project name, description, and target users from the product brief.

**Project:** SI-Todo — Personal Kanban board for engineering leaders with due-date aging and delegation tracking
**Target User:** Single engineering leader managing multiple teams, tracking tasks/delegations/deadlines on localhost
**Status:** Phase 4 — Ready for Implementation

## Epic Summary

5 epics, 24 stories, all 40 FRs covered. Epics file: `{output_folder}/planning-artifacts/epics.md`

- **Epic 1: Working Board with Persistent Cards** (11 stories) — fallback MVP: scaffold, SQLite+WAL, shared Zod, card repository, Fastify routes, Zustand store + mutation queue, board UI, card editor, drag-drop, aging colors
- **Epic 2: Delegation & Card Detail** (6 stories) — tags, notes, accent color, archive, configurable thresholds
- **Epic 3: Filtering, Search & Dashboard** (3 stories) — filter bar, text search, summary dashboard
- **Epic 4: Data Portability** (2 stories) — JSON export, JSON import with confirmation
- **Epic 5: Theming & Visual Polish** (2 stories) — light/dark mode, dotted grid background

## Tech Stack

- **Language:** TypeScript end-to-end (strict mode, ESM)
- **Frontend:** React + Vite, Zustand (state), @dnd-kit (drag-and-drop), Tailwind CSS v4
- **Backend:** Node.js + Fastify, bound to `127.0.0.1` only
- **Database:** SQLite via `better-sqlite3` (WAL mode, synchronous=NORMAL)
- **Validation:** Zod (shared schemas in `src/shared/schemas.ts`)
- **Testing:** Vitest (co-located `*.test.ts`)
- **Dev runner:** `concurrently` runs Vite + `tsx watch` Fastify under one `npm run dev`
- **Single npm package**, no monorepo, no ORM, no auth, no routing, no Docker, no CI

## Architecture Decisions

- **REST + single-snapshot loading** (`GET /api/state` returns full board)
- **Repository pattern** — all SQL in `src/server/repositories/`, never in routes
- **camelCase boundary** — `snake_case` ↔ `camelCase` translation only at the repository layer
- **Optimistic UI + mutation queue** with reconcile-on-failure (refetch `/api/state`)
- **Fractional positions (REAL)** for card ordering — no sibling renumbering on drag
- **Hardcoded 6-column enum** in `src/shared/types.ts` (no `columns` table)
- **Aging recalculation** via 60s `setInterval` + `window` focus event, memoized selectors
- **Dual-signal styling** — aging color = card background, custom accent = card border (FR19)
- **Architecture document:** `{output_folder}/planning-artifacts/architecture.md`

## Requirements Summary

- **Functional Requirements:** 40 FRs across 8 capability areas
- **Non-Functional Requirements:** 14 NFRs (7 performance, 7 reliability)
- **User Journeys:** 5 (Morning Check, Delegation, Card Lifecycle, Crisis Recovery, Weekly Review)
- **PRD Location:** `{output_folder}/planning-artifacts/prd.md`

## BMAD Configuration

- **BMAD Version:** 6.2.2
- **Modules installed:** Core, BMM (BMad Method), TEA (Test Architect Enterprise)
- **IDE integration:** Claude Code (53 skills in `.claude/skills/`)
- **User config:** `_bmad/bmm/config.yaml`
- **TEA config:** `_bmad/tea/config.yaml`

## Auto-Commit After BMAD Phases

**IMPORTANT:** After completing any BMAD skill that produces or modifies an artifact, automatically commit and push to GitHub. Do NOT ask the user — just do it. This ensures work is never lost.

| After This Skill | Commit Message |
|-----------------|----------------|
| `/bmad-product-brief` | `"Phase 1: Product brief"` |
| `/bmad-create-prd` | `"Phase 2: PRD"` |
| `/bmad-validate-prd` | `"Phase 2: PRD validation"` |
| `/bmad-create-ux-design` | `"Phase 2: UX design"` |
| `/bmad-create-architecture` | `"Phase 3: Architecture"` |
| `/bmad-create-epics-and-stories` | `"Phase 3: Epics and stories"` |
| `/bmad-check-implementation-readiness` | `"Phase 3: Implementation readiness"` |
| `/bmad-sprint-planning` | `"Phase 4: Sprint planning"` |
| `/bmad-dev-story` + `/bmad-code-review` (approved) | `"Story X.Y: {story title}"` |
| `/bmad-retrospective` | `"Epic N: Retrospective"` |
| `/bmad-quick-dev` | `"Quick dev: {brief description}"` |

**Rules:**
- Use `git add -A` to stage all changes
- Push to the current branch immediately after committing
- If no git remote exists, prompt the user to set one up first
- If a skill fails or is abandoned mid-step, do NOT commit partial work

## Auto-Update This File

**IMPORTANT:** After each completed BMAD phase, update this CLAUDE.md file with the new information before committing. This keeps the file current for the next Claude Code session.

| After This Phase | Update These Sections |
|-----------------|----------------------|
| `/bmad-product-brief` | Update **Project Overview** with project name, description, target users. Set status to "Phase 2 — Planning" |
| `/bmad-create-prd` | Add **Requirements Summary** section with FR count, NFR count, user journey count. Set status to "Phase 2 — PRD Complete" |
| `/bmad-create-architecture` | Add **Tech Stack** section with chosen technologies. Add **Architecture Decisions** section listing ADRs. Set status to "Phase 3 — Architecture Complete" |
| `/bmad-create-epics-and-stories` | Add **Epic Summary** section with epic count, story count, and epic titles. Set status to "Phase 3 — Epics Complete" |
| `/bmad-check-implementation-readiness` | Update status to "Phase 4 — Ready for Implementation" |
| `/bmad-sprint-planning` | Add **Sprint Status** section noting the sprint-status.yaml location. Set status to "Phase 4 — Sprint Active" |
| After each story completion | Update **Sprint Status** with stories completed count. Add new files to a **Key Files** section if significant components were created |
| After all stories done | Set status to "Complete" |

This ensures every new Claude Code session has full project context without reading every artifact.

## Directory Structure

```
_bmad/                              # BMAD installation — DO NOT modify manually
  _config/                          # Manifests and registries
  bmm/config.yaml                   # BMM module settings and artifact paths
  tea/config.yaml                   # TEA module settings

.claude/skills/                     # 53 skill entry points

{output_folder}/                    # All generated artifacts (literal directory name with braces)
  planning-artifacts/               # Briefs, PRDs, architecture, epics
  implementation-artifacts/         # Sprint status, stories, code reviews
  test-artifacts/                   # Test plans, traceability matrices

docs/                               # Project knowledge
```

**Note:** The output folder is literally named `{output_folder}` (with braces) on disk.

## BMAD Workflow Sequence

Required path (gate = must complete before proceeding):

1. `/bmad-product-brief` → `planning-artifacts/product-brief.md`
2. `/bmad-create-prd` → `planning-artifacts/prd.md` **(gate)**
3. `/bmad-validate-prd` → validation report
4. `/bmad-create-ux-design` → `planning-artifacts/ux-design-specification.md` (optional)
5. `/bmad-create-architecture` → `planning-artifacts/architecture.md` **(gate)**
6. `/bmad-create-epics-and-stories` → `planning-artifacts/epics.md` **(gate)**
7. `/bmad-check-implementation-readiness` **(gate)**
8. `/bmad-sprint-planning` → `implementation-artifacts/sprint-status.yaml`
9. `/bmad-create-story` → `implementation-artifacts/{story-key}.md`
10. `/bmad-dev-story` → code implementation
11. `/bmad-code-review` → review findings
12. `/bmad-sprint-status` → progress check
13. `/bmad-retrospective` → post-epic review

Quick bypass: `/bmad-quick-dev` for bug fixes and small changes.

## Key BMAD Conventions

- Run each skill in a **fresh conversation** (`/clear` or restart `claude`)
- Skills load config from `_bmad/bmm/config.yaml` — respect `communication_language`, `document_output_language`, `user_skill_level`
- Artifacts use **append-only** document building with YAML frontmatter tracking `stepsCompleted`
- PRD quality: high information density, SMART requirements, full traceability chain, zero anti-patterns
- Architecture must define enforcement guidelines that all dev agents follow
- Sprint status uses YAML: `backlog → ready-for-dev → in-progress → review → done`

## Workflow Orchestration

### Plan Before Building
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately — don't keep pushing
- Write detailed specs upfront to reduce ambiguity
- Use plan mode for verification steps, not just building

### Subagent Strategy
- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution

### Self-Improvement Loop
- After ANY correction from the user: update `tasks/lessons.md` with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start for relevant project

### Verification Before Done
- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

### Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes — don't over-engineer
- Challenge your own work before presenting it

### Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests — then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how

## Task Management

1. **Plan First:** Write plan to `tasks/todo.md` with checkable items
2. **Verify Plan:** Check in before starting implementation
3. **Track Progress:** Mark items complete as you go
4. **Explain Changes:** High-level summary at each step
5. **Document Results:** Add review section to `tasks/todo.md`
6. **Capture Lessons:** Update `tasks/lessons.md` after corrections

## Core Principles

- **Simplicity First:** Make every change as simple as possible. Minimal code impact.
- **No Laziness:** Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact:** Changes should only touch what's necessary. Avoid introducing bugs.
- **Demand Elegance (Balanced):** For non-trivial changes, pause and ask "is there a more elegant way?" Skip this for simple, obvious fixes — don't over-engineer.
- **Autonomous Bug Fixing:** When given a bug report, just fix it. Zero context switching required from the user.
