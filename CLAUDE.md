# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the App

No build step required — open `index.html` directly in a browser:

```bash
open index.html
# or serve locally
python3 -m http.server 8000
```

## Architecture

This is a **zero-dependency, single-file SPA** (`index.html`, ~1500 lines) with all HTML, CSS, and JavaScript embedded inline. No build tools, no frameworks, no transpilation.

**Persistence:** Browser `localStorage` under key `fitplan_v3`. Exercise completion is stored as a flat object with keys like `w1-Mon-e0`, `w3-Fri-e2` (format: `w{week}-{day}-e{exerciseIndex}`). `getState()` / `saveState(s)` are the only read/write functions for this store. `resetAll()` clears the entire key.

**State flow:**
1. User interaction (e.g., checkbox click) → `toggleEx(week, day, exerciseIndex)`
2. Toggles the boolean in localStorage
3. `refreshAll()` re-renders all pages and updates the sidebar progress bar
4. Each page has its own builder: `buildToday()`, `buildWeekTabs()` + `buildWeekContent()`, `buildMeals()`, `buildRecipes()`, `buildPuppy()`, `buildProgress()`

**Navigation:** 6 pages (Today, Workouts, Meals, Recipes, Puppy Schedule, Progress) rendered by `showPage(name)` — only one `.page` div is visible at a time. Desktop uses a sticky sidebar; mobile uses a bottom nav bar (breakpoint: 640px). Some builders (`buildProgress`, `buildRecipes`, `buildPuppy`) are lazy — only called when their page becomes visible.

**Key data structures (JS constants at the top of the `<script>` block):**
- `SCHEDULE` — 7-day workout plan reused across 4 weeks; each exercise has per-week progressive sets (`sets: { 1:'4×6', 2:'4×7', ... }`)
- `MEALS` — 7-day meal rotation (breakfast/lunch/dinner/snack) with emoji + text per slot
- `RECIPES` — keyed by category (`'Sunday Prep'`, `'Breakfast'`, `'Lunch & Dinner'`, `'Quick Fixes'`); each recipe has ingredients, steps, tips, and optional images
- `WEEKDAY_SCHEDULE` / `WEEKEND_NOTES` — daily planner data for the Puppy Schedule page

**In-page state (not persisted):** `activeWeek` (1–4), `activeMealDay` (0–6), `activeRecipeCat` (string) — these reset on page reload.

## CSS Design System

All styles are in the embedded `<style>` block. Key custom properties:
- Base colors: `--orange`, `--green`, `--blue`, `--purple`, `--grey` (each has a `-light` variant, orange also has `-mid`)
- Layout: `--border`, `--text`, `--text-soft`, `--bg`, `--white`
- Shared: `--radius: 14px`, `--shadow`
- Workout badge types map to dot classes: `.dot-strength` (orange), `.dot-cardio` (blue), `.dot-recovery` (green), `.dot-rest` (grey), `.dot-endurance` (purple)

## Images

Images live under `images/` with subdirectories: `workouts/`, `exercises/`, `meals/`, `recipes/`. Referenced as relative paths. All `<img>` tags include `onerror="this.parentElement.style.display='none'"` (or `this.style.display='none'`) so the app degrades gracefully when images are missing.
