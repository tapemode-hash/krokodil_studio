# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Web application for managing a children's art studio ("Krokodil") in Arkhangelsk, Russia. Combines a public landing page with an internal CRM for teachers and an administrator. Hosted at `tapemode-hash.github.io/krokodil_studio/`.

## Development

No build tools, package managers, or dependencies. Everything is plain HTML/CSS/JS.

```bash
# Recommended: local HTTP server (required for crypto.subtle / admin login)
python3 -m http.server 8000
# then open http://localhost:8000

# Alternative: open directly in browser (admin login will NOT work via file://)
open index.html
```

## Deployment

Push to `main` → GitHub Pages automatically deploys `index.html`. Changes go live within minutes.

```bash
git add index.html
git commit -m "description"
git push origin main
```

Branch workflow: `fix/...` / `feat/...` / `claude/...` → PR → squash merge → `main`.

## Architecture

The entire application is a **single file: `index.html`** (~7500+ lines):
- `<style>` — all CSS
- `<body>` — all markup: pages, modals, overlays, login screen
- `<script>` — all JS: constants, data, functions, event handlers

`krokodil_studio.html` is a backup copy, identical to `index.html`. Keep both in sync when making changes.

### Navigation

```js
let currentPage = ''
showPage(id)    // hides all .page, shows target, calls its render function
```

No `history.pushState`. URL never changes. All pages are `<div id="page-*" class="page">` in the DOM, hidden by default, shown with `.active`.

### Data Storage

All data lives in **global JS variables** and is persisted to `localStorage` with prefix `krok_`.

```js
lsLoad(key, def)   // read from localStorage with fallback
lsSave(key, val)   // write to localStorage
persist()          // saves all ~29 variables at once — call after every mutation
```

After any data mutation: update the variable → call `persist()` → call the relevant render function.

Current `DATA_VERSION = 2`. Auto-migration runs on load when stored version is lower.

### Rendering

No Virtual DOM or reactivity. Every change triggers a full DOM re-render of the relevant section via its dedicated `renderPageName()` function. `renderAll()` re-renders everything.

### Authentication / Roles

```js
let currentRole = 'guest' | 'parent' | 'admin'
let _parentStudentIds = []   // IDs of children for the logged-in parent

const ADMIN_PW_H = '9511aa000732a111985d7c746e454b746d7e4cd69c5c941ed0d5dac9c99c228b'
// SHA-256 of admin password; crypto.subtle requires localhost or HTTPS

const guestPages  = ['dashboard','schedule','teachers','gallery']
const parentPages = ['dashboard','schedule','teachers','gallery','attendance']
// admin: all pages
```

Parent login: phone number from `students[].phone` — the system finds matching children.

## Data Model

### localStorage keys (prefix `krok_`)

| Key | Contents |
|---|---|
| `krok_teachers` | Teacher array |
| `krok_students` | Student array |
| `krok_schedule` | Schedule group array |
| `krok_attendance` | `{ 'YYYY-MM-DD': { attKey: 'p'\|'a'\|'trial' } }` |
| `krok_enrollments` | `{ sid: [scId, ...] }` — permanent group enrollments |
| `krok_dateEnrollments` | `{ sid: ['YYYY-MM-DD', ...] }` — date-specific enrollments |
| `krok_dayAssignments` | `{ 'YYYY-MM-DD': { sid: scId } }` — one-day group assignments |
| `krok_subscriptions` | `{ sid: [{ groupName, total, used, price, createdAt }, ...] }` |
| `krok_lessonCosts` | `{ 'YYYY-MM-DD': { attKey: price, attKey_via_sub: bool, ... } }` |
| `krok_globalLessonPrice` | Default lesson price (500 ₽) |

### Attendance key (attKey) format

- `String(sid)` — student in zero or one group
- `"sid:scId"` — student temporarily assigned to a group via `dayAssignments`

When a student with a plain key gets assigned to a group, `attKey` migrates to composite form, and reverts on removal.

## Key Business Rules

**Active student**: has an attendance record (`p`, `a`, or `trial`) or a `dateEnrollment` entry within the last 6 months. Use `isStudentActive(sid)` to check. All current-state metrics (`renderReports`, `renderReportTable`, retention, LTV) filter to active students only. Do **not** apply this filter to historical/financial data.

**Subscriptions**: `getActiveSub(sid)` returns the last subscription with `used < total`. On each attended lesson with an active sub, increment `used` and set `lessonCosts[date][attKey_via_sub] = true`.

**Teacher attribution** in financial metrics: check `enrollments[sid]` first; fall back to `dayAssignments[dateStr][sid]` if empty; use composite `sid:scId` key for day-assigned students.

## Design System

```css
:root {
  --cream: #FFF8F0;  --warm: #FFFDF9;
  --coral: #FF8C69;  /* primary accent */
  --mint:  #B8F0D4;  --mint-d: #5ECFA0;
  --lav:   #D4C4F0;  --lav-d:  #9B84D8;
  --sky:   #C4E4F8;  --sky-d:  #5BAED6;
  --ink:   #3A2E2E;  --ink-l:  #6B5E5E;
}
```

Fonts: **Nunito** (UI body, weights 300/400/600/700) + **Caveat** (decorative/headings, weights 400/600/700) via Google Fonts.

Responsive breakpoints: `480px`, `768px`, `900px`, `print`.

Modal pattern: all modals have `id="m-*"` and use `openModal(id)` / `closeModal(id)`.
