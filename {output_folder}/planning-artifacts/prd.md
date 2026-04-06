---
stepsCompleted: [step-01-init, step-02-discovery, step-02b-vision, step-02c-executive-summary, step-03-success, step-04-journeys, step-05-domain-skipped, step-06-innovation-skipped, step-07-project-type, step-08-scoping, step-09-functional, step-10-nonfunctional, step-11-polish]
inputDocuments:
  - docs/planning-artifacts/product-brief-si-todo.md
  - docs/planning-artifacts/product-brief-si-todo-distillate.md
documentCounts:
  briefs: 2
  research: 0
  brainstorming: 0
  projectDocs: 0
workflowType: 'prd'
classification:
  projectType: Full-Stack Web App
  domain: General / Productivity
  domainComplexity: Low
  productComplexity: Medium
  projectContext: Greenfield
---

# Product Requirements Document - SI-Todo

**Author:** Saidul
**Date:** 2026-04-05

## Executive Summary

SI-Todo is a personal Kanban board web application built for a single engineering leader who manages multiple teams and workstreams. It solves a specific problem: the mental overhead of tracking personal tasks, delegations, and deadlines when corporate tools are either blocked, overkill, or abandoned. The real competitor is the neglected text file — and SI-Todo wins by being the tool that actually works because it lives outside the corporate stack.

The app runs on localhost with a persistent backend — no deployment, no authentication, no IT ticket. A six-column board (Backlog, To Do, In Progress, Delegated, Blocked, Done) provides a single-screen command center. Every change auto-saves to disk, surviving server restarts, VDI shutdowns, and browser closures.

### What Makes This Special

Two signature features define SI-Todo as a leadership tool rather than a generic task app:

1. **First-class Delegated column with assignee tracking.** Leaders delegate constantly, and no lightweight personal tool handles this well. SI-Todo makes "who did I assign this to, and when is it due?" a first-class workflow — filterable by assignee for instant 1:1 prep or follow-up checks.

2. **Due-date aging as a visual heat map.** Cards automatically shift from green (7+ days) → yellow (3–7 days) → red (<3 days) → pulsing red (overdue). The board tells the truth even when you don't ask — opening it in the morning instantly shows what's overdue, at risk, and on track without reading a single date.

These combine with user-customizable card colors (applied as border/accent, coexisting with aging backgrounds), priority levels (P1/P2/P3), filtering/search, and a summary dashboard to create a tool that turns passive task tracking into active situational awareness.

## Project Classification

| Dimension | Value |
|-----------|-------|
| **Project Type** | Full-Stack Web App (React SPA + lightweight backend) |
| **Domain** | General / Productivity |
| **Domain Complexity** | Low — no regulated industry, no compliance requirements |
| **Product Complexity** | Medium — real-time visual state, drag-and-drop, dual-signal color system, persistent auto-save |
| **Project Context** | Greenfield — new product, no existing codebase |

## Success Criteria

### User Success

- **Zero dropped balls.** No task, delegation, or commitment falls through the cracks because it wasn't visible on the board.
- **Instant situational awareness.** Opening the board immediately communicates what's overdue (red/pulsing), at risk (yellow), and on track (green) — without reading individual dates or descriptions.
- **Effortless delegation tracking.** "What did I delegate to Ahmed?" is answered in one filter action, not a mental search through memory or chat history.
- **No missed deadlines.** Due-date aging creates passive urgency signals that surface risk days before it becomes a crisis.

### Business Success

- **Daily habit formation.** The board becomes the first tab opened each morning and the last checked before end of day — measured by consistent daily use over weeks.
- **Personal indispensability.** The user stops maintaining parallel tracking systems (text files, sticky notes, pinned Slack messages) because the board fully replaces them.
- **Zero context-switching cost.** The tool requires no login, no loading screens, no configuration — open browser, see truth.

### Technical Success

- **Reliable persistence.** Data survives server restarts, VDI shutdowns, and browser closures without exception. Zero data loss incidents.
- **Responsive interaction.** Board operations (drag-and-drop, filtering, card creation) feel instant and fluid. No perceptible lag during normal use.
- **Zero-infrastructure operation.** The app runs entirely on localhost with no external dependencies, no network exposure, and no setup beyond starting the server.

### Measurable Outcomes

| Outcome | Target |
|---------|--------|
| Dropped commitments after adoption | Zero |
| Missed deadlines due to lack of visibility | Zero |
| Daily board usage (morning check) | Every workday |
| Parallel tracking systems still in use | None — board is single source of truth |
| Data loss incidents | Zero |
| Time from server start to usable board | Seconds, not minutes |

## Product Scope

### MVP Strategy

**MVP Approach:** Problem-Solving MVP — the minimum feature set that makes the neglected text file obsolete. The product succeeds when it's more useful than the current alternative (scattered notes, Slack messages, mental tracking) on day one.

**Resource Requirements:** Single developer with full-stack capability (React + lightweight backend). No design team needed — clean, functional UI over polished visuals. No DevOps — localhost deployment eliminates infrastructure.

### MVP Feature Set (Phase 1)

All five primary user journeys are fully supported by the MVP feature set:

| Journey | MVP Support |
|---------|------------|
| Morning Board Check | Aging colors + board layout + dashboard summary |
| Delegation Workflow | Assignee field + Delegated column + filter by assignee |
| Card Lifecycle | Card CRUD + drag-and-drop + archive/delete |
| Crisis Recovery | Pulsing red urgency + card editing + JSON export |
| Weekly Review | Archive + delete + reprioritize + filter |

**Must-Have Capabilities:**
- 6-column Kanban board (Backlog, To Do, In Progress, Delegated, Blocked, Done) with drag-and-drop
- Card CRUD: title, description, assignee, due date, priority (P1/P2/P3), tags
- Due-date aging with configurable thresholds (green 7+ days → yellow 3–7 days → red <3 days → pulsing red overdue)
- User-customizable card accent colors (border/accent, coexisting with aging background)
- Filter/search by assignee, priority, due date, tag
- Summary dashboard: total items, overdue count, by assignee, by priority
- Light/dark mode toggle
- Persistent backend storage (file-based or SQLite) on localhost with auto-save
- Card archive/delete to prevent Done column from growing unbounded
- JSON export/import for backup and data portability
- Desktop browser UI only (no mobile)
- No authentication

### Phase 2 — Growth Features

- Recurring tasks for regular responsibilities
- Quick-capture via keyboard shortcut
- 1:1 prep view — filter delegated items by assignee for meeting prep
- Archive and history — searchable log of completed items
- Weekly digest export for status reporting

### Phase 3 — Vision

- Optional cloud sync across devices
- Mobile-friendly quick-capture widget
- Customizable board layouts or additional columns
- Advanced card color rule system (auto-coloring by rule, not just manual)

### Risk Mitigation

**Technical Risks:**
- *Drag-and-drop complexity* — Use a proven library (dnd-kit or similar) rather than building from scratch. Deferred to architecture decision.
- *Due-date aging performance* — Client-side timer recalculation across 30+ cards. Mitigated by efficient re-render strategy (deferred to architecture).
- *Data persistence reliability* — The core promise. Mitigated by backend-first storage (not localStorage), plus JSON export as manual backup.

**Market Risks:**
- *Habit formation* — Single-user tools risk abandonment. Mitigated by due-date aging creating passive pull (the board demands attention via color) and the morning check ritual.
- *Scope creep toward team tool* — Resist. The product's strength is being radically single-player. Multi-user features would dilute the value proposition.

**Resource Risks:**
- *Single developer* — If velocity drops, the MVP feature set is already lean. The dashboard is MVP-critical for instant situational awareness (user-confirmed).
- *Fallback MVP* — If forced to ship even smaller: board + cards + drag-and-drop + aging + persistence. No filters, no dashboard, no export. Supports Journey 1 (Morning Check) and Journey 3 (Card Lifecycle) only.

## User Journeys

### Journey 1: The Morning Board Check

**Persona:** Saidul — engineering leader managing three teams across infrastructure, platform, and data workstreams. Juggles 30+ active items at any given time. Has been burned before by missed follow-ups that only surfaced in a 1:1 when it was too late.

**Opening Scene:** It's 8:15 AM. Saidul opens his laptop, launches the browser, and his first tab loads `localhost:3000`. The board appears instantly — no login, no loading spinner. His eyes scan the columns before his coffee is ready.

**Rising Action:** Three cards are pulsing red — overdue. Two are in Delegated (Ahmed's deployment fix was due yesterday, Priya's API spec was due two days ago). One is in his own In Progress column (the architecture review he kept pushing). Five more cards glow yellow — due within three days. The rest are green. Without reading a single date, Saidul knows exactly where the fires are.

**Climax:** He drags the architecture review to the top of his To Do column and bumps it to P1. He makes a mental note to ping Ahmed and Priya in standup. He spots a Blocked card he'd forgotten about — the vendor access request that's been sitting for a week. He drags it back to To Do and adds a note to escalate.

**Resolution:** In under two minutes, Saidul has a complete picture of his day. No scrolling through Slack, no checking a text file, no trying to remember what he forgot. The board told him the truth before he even asked.

---

### Journey 2: The Delegation Workflow

**Opening Scene:** Saidul is in a 1:1 with his tech lead, Fatima. She mentions the logging pipeline needs a redesign. Saidul agrees to delegate it and wants to track the commitment.

**Rising Action:** After the meeting, Saidul opens the board and clicks "Add Card." He types the title ("Redesign logging pipeline"), assigns it to Fatima, sets a due date two weeks out, tags it `infrastructure` and `P2`, and drops it directly into the Delegated column. The card appears with a green background — plenty of time.

**Climax:** A week later, Saidul is prepping for his next 1:1 with Fatima. He filters the board by assignee: "Fatima." Three cards appear — the logging redesign (now yellow, one week left), a monitoring dashboard task (green), and a completed incident postmortem (in Done). He has an instant agenda without digging through notes or chat history.

**Resolution:** In the 1:1, Saidul asks Fatima directly about the logging redesign because the board surfaced it. Fatima mentions she's blocked on access permissions. Saidul moves the card to Blocked, adds a note, and creates a new card for himself to escalate the access request. Nothing fell through the cracks because the delegation was tracked from the moment it was made.

---

### Journey 3: Card Lifecycle & Board Management

**Opening Scene:** Saidul has just come out of a planning meeting with six new action items. Some are his, some are delegations, some are blocked pending decisions from leadership.

**Rising Action:** He rapidly creates six cards:
- "Finalize Q3 headcount proposal" → To Do, P1, due Friday
- "Review Ahmed's promotion packet" → To Do, P2, due next week
- "Data team migration plan" → Delegated to Raj, P2, due in 10 days
- "Budget approval for new tooling" → Blocked, tagged `waiting-on-leadership`
- "Update oncall rotation" → Backlog, P3, no due date
- "Prepare board presentation slides" → In Progress, P1, due tomorrow

Each card gets the right column, priority, assignee (if delegated), and due date. The board immediately reflects urgency — the presentation slides glow yellow, everything else is green.

**Climax:** Over the next two weeks, cards move through the board. The presentation slides go from In Progress → Done. The headcount proposal moves To Do → In Progress → Done. The budget card gets unblocked and moves Blocked → To Do → In Progress. Saidul archives completed cards from Done to keep the board clean.

**Resolution:** The board is a living artifact of Saidul's work. Cards flow left to right, Done items get archived, and the board never becomes a graveyard of stale items. Every card either moves forward or gets consciously deprioritized — nothing rots silently.

---

### Journey 4: Crisis Recovery — The Missed Deadline

**Opening Scene:** It's Thursday afternoon. Saidul opens his board and sees a pulsing red card in Delegated — "Security audit remediation" assigned to Omar, due two days ago. Saidul realizes he never followed up after assigning it last sprint.

**Rising Action:** Saidul checks with Omar via Slack. Omar says he deprioritized it because another P1 came in. The remediation is half-done. Saidul needs to triage: the security team expects it by end of week.

**Climax:** Saidul moves the card from Delegated back to In Progress — he'll pair with Omar to finish it. He updates the due date to Friday, bumps it to P1, and adds a note: "Pairing with Omar, security team expects EOW." He scans the rest of his board for anything else he can deprioritize to make room. He moves a P2 card back to Backlog to free up bandwidth.

**Resolution:** The crisis was caught because the board made it visually impossible to ignore — a pulsing red card demands attention. The recovery was quick because all the context (assignee, original due date, priority) was on the card. Saidul also exports a JSON backup of the board state before making major triage changes — a safety net in case he needs to undo his reprioritization.

---

### Journey 5: The Weekly Review

**Opening Scene:** It's Friday at 4 PM. Saidul sets aside 15 minutes for his weekly board review — a ritual that's become as automatic as the morning check.

**Rising Action:** He starts with the Done column. Seven cards completed this week. He scans them for a quick sense of accomplishment and progress, then archives all seven to keep the board clean. Next, he checks the Delegated column — filters by assignee to see who has what. Two items for Ahmed (both on track), one for Fatima (yellow, due Monday — he makes a mental note to check in), one for Raj (green, not due for a week).

**Climax:** Saidul turns to the Backlog and To Do columns. Three cards in Backlog haven't moved in two weeks — they're stale. He deletes one that's no longer relevant, reprioritizes another from P3 to P2 (it's become more urgent), and leaves the third as-is. In To Do, he reorders cards for next week's priorities: the two P1s go to the top, a P2 that's turning yellow gets attention. He spots a card with no due date — adds one for next Thursday to make sure it doesn't drift indefinitely.

**Resolution:** In 15 minutes, Saidul has a clean board heading into next week. Completed work is archived, stale items are pruned, priorities are recalibrated, and every card has a clear owner and timeline. He closes the browser knowing Monday morning's board check will be clean and actionable. The weekly review is what keeps the board honest over time — without it, entropy wins.

---

### Journey Requirements Summary

| Journey | Key Capabilities Revealed |
|---------|--------------------------|
| Morning Board Check | Instant load, visual heat map scanning, due-date aging colors, drag-and-drop reordering |
| Delegation Workflow | Card creation with assignee, Delegated column, filter by assignee, card notes/updates |
| Card Lifecycle | Rapid card creation, column drag-and-drop, archive/delete from Done, priority management |
| Crisis Recovery | Visual urgency (pulsing red), card editing (reassign, reprioritize, update due date), JSON export |
| Weekly Review | Archive bulk actions, Backlog/Done column scanning, reprioritization, stale item cleanup, delete cards |

## Web App Technical Requirements

### Application Architecture

**Type:** Single-Page Application (SPA) on localhost

- Single-screen Kanban board — all interaction happens on one view
- Client-side state management for board operations (drag-and-drop, filtering, card editing)
- Backend API for CRUD operations and persistent storage
- Auto-save on every mutation — no manual save action

**Real-Time Behavior:**
- Due-date aging requires periodic color recalculation as time passes
- No multi-user real-time (WebSocket not needed)
- Client-side timer or re-render on window focus to update aging colors
- Board state reflects current aging status whenever the user is looking at it

### Browser Support

| Browser | Support Level |
|---------|--------------|
| Chrome (latest) | Primary — fully supported |
| Edge (latest) | Primary — fully supported |
| Firefox | Not supported in V1 |
| Safari | Not supported in V1 |

Chromium-based browsers only — simplifies CSS and API surface.

### Accessibility

- Basic keyboard navigation: tab through cards, enter to open/edit, escape to close
- Reasonable color contrast in both light and dark modes
- Due-date aging colors must be distinguishable (green/yellow/red are inherently high-contrast)
- No formal WCAG compliance target for V1

### Explicitly Out of Scope (Web App)

- Responsive/mobile layouts — desktop viewport with mouse and keyboard only
- SEO — localhost, no public URL, no indexing
- SSR — SPA is sufficient, no server-side rendering needed
- Authentication — no auth middleware, session management, or login UI
- CORS — frontend and backend on same localhost origin
- SSL/TLS — localhost only, no network exposure
- CDN or asset optimization — local serving only

### Theming

- Light/dark mode via CSS custom properties or theme provider, toggled via UI control
- Dotted grid background pattern available in light mode

## Functional Requirements

### Board Structure & Layout

- **FR1:** User can view a Kanban board with six fixed columns: Backlog, To Do, In Progress, Delegated, Blocked, Done
- **FR2:** User can view all cards within each column in a vertically scrollable list
- **FR3:** User can reorder cards within a column via drag-and-drop
- **FR4:** User can move cards between columns via drag-and-drop

### Card Management

- **FR5:** User can create a new card with title, description, assignee, due date, priority (P1/P2/P3), and tags
- **FR6:** User can edit any field on an existing card
- **FR7:** User can delete a card permanently
- **FR8:** User can archive a card from the Done column
- **FR9:** User can assign a card directly to a specific column during creation
- **FR10:** User can add free-text notes to a card

### Due-Date Aging System

- **FR11:** System automatically displays cards with a green background when the due date is 7+ days away
- **FR12:** System automatically displays cards with a yellow background when the due date is 3–7 days away
- **FR13:** System automatically displays cards with a red background when the due date is fewer than 3 days away
- **FR14:** System automatically displays cards with a pulsing red background when the due date has passed
- **FR15:** Cards with no due date display with a neutral background (no aging color)
- **FR16:** User can configure the aging threshold values (days for green, yellow, red transitions)
- **FR17:** System recalculates aging colors in real time while the board is open

### Card Customization & Visual System

- **FR18:** User can assign a custom accent color to any card
- **FR19:** Custom accent colors display as card border/accent, coexisting with aging background colors without conflict
- **FR20:** Cards display their priority level (P1/P2/P3) visually
- **FR21:** Cards display their assigned tags visually

### Filtering & Search

- **FR22:** User can filter the board by assignee
- **FR23:** User can filter the board by priority level
- **FR24:** User can filter the board by due date range
- **FR25:** User can filter the board by tag
- **FR26:** User can combine multiple filters simultaneously
- **FR27:** User can clear all active filters to return to the full board view
- **FR28:** User can search cards by text content (title, description)

### Dashboard & Summary

- **FR29:** User can view a summary dashboard showing total active card count
- **FR30:** User can view a count of overdue cards in the dashboard
- **FR31:** User can view a breakdown of cards by assignee in the dashboard
- **FR32:** User can view a breakdown of cards by priority level in the dashboard

### Data Persistence & Portability

- **FR33:** System automatically saves all changes to backend storage without manual save action
- **FR34:** System persists all board data across server restarts
- **FR35:** System persists all board data across browser closures
- **FR36:** User can export the complete board state as a JSON file
- **FR37:** User can import a previously exported JSON file to restore board state

### Appearance & Theming

- **FR38:** User can toggle between light mode and dark mode
- **FR39:** Light mode displays a dotted grid background option
- **FR40:** User's theme preference persists across sessions

## Non-Functional Requirements

### Performance

- **NFR1:** Board initial load completes in under 2 seconds with up to 50 active cards
- **NFR2:** Drag-and-drop card movement responds in under 100ms (feels instant)
- **NFR3:** Filter and search results display in under 300ms
- **NFR4:** Card creation, editing, and deletion reflect on the board in under 200ms
- **NFR5:** Auto-save completes in the background without blocking user interaction
- **NFR6:** Due-date aging color recalculation does not cause visible UI stutter or delay
- **NFR7:** Dashboard summary updates without perceptible delay after board changes

### Reliability & Data Integrity

- **NFR8:** Zero data loss across server restarts — all board state is recoverable
- **NFR9:** Zero data loss across browser closures — no pending changes are lost
- **NFR10:** Zero data loss across VDI session shutdowns — backend persists to disk, not memory
- **NFR11:** Auto-save triggers on every user mutation — no manual save action required
- **NFR12:** JSON export produces a complete, valid, re-importable snapshot of all board data
- **NFR13:** JSON import restores board state fully without data corruption or loss
- **NFR14:** Backend storage is resilient to unclean shutdown (process kill, power loss) — data corruption must not occur from incomplete writes
