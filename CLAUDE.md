# CLAUDE.md — SFOA Website

## MCP Tool Usage

MCP tools are the default — use them unless there is a specific reason not to. Before acting on any prompt, consider which tool is the best fit:

- **code-review-graph** (`mcp__code-review-graph__*`) — preferred for structural questions: impact analysis, blast radius, call graph traversal, architecture overview, finding what a change will affect. Use before editing to scope the blast radius; use after to verify nothing unexpected was touched.
- **SDL-MCP** (`mcp__sdl-mcp__*`) — preferred for precise symbol navigation: finding where a function is defined, listing all references to a symbol, or confirming a function signature.
- **Native tools** (Read, Grep, Glob) — fall back to these when the question is simpler than what an MCP tool is designed for, or when the relevant MCP server doesn't cover the file type.

The choice should be driven by the nature of the task, not by habit. Do not default to Read/Grep when a graph query or symbol lookup would give a richer, more accurate answer with less context consumption.

---

## Project Overview

*(Fill in: high-level description of what this website is, who it's for, and what it does.)*

---

## Design System

### Colors

- **Primary greens (SFOA brand):**
  - 900 `#1A5632` — primary brand / hero
  - 50  `#F0FBF4` — pale surfaces
  - Fill in intermediate ramp (700/500/300/100) as needed
- **Neutrals:** `#1B1B28` (900) → `#FAFAFA` (50)
- **Accent palette (optional, from sibling projects):**
  - Discussion `#2D6A4F`
  - Video `#3A7CA5`
  - Activity `#E07B39`
  - Follow-up `#7B5EA7`

### Typography

- **Display / headings:** Zodiak (serif, via Fontshare) — self-host where possible
- **Body:** Work Sans (sans-serif, via Google Fonts) — self-host where possible
- **Fluid scaling:** use `clamp()` throughout for type sizes
- **Spacing scale:** base unit 4px (`--space-1` = 4px → `--space-32` = 128px)

### CSS Organization

Design tokens → reset → layout → components. Label each section with a comment block. Mobile-first responsive: hamburger nav / stacked layout on small screens, full layout on desktop.

---

## CSS Gotchas

### Global `p { max-width: 72ch }` reset

A common readability reset applies `max-width: 72ch` to `p, li, figcaption` globally. This can cause centering failures: if a `<p>` is narrower than its flex/grid container, it will be left-aligned within the container even when the container has `text-align: center` or `justify-content: center`. Fix with:
```css
.your-container p { max-width: none; }
```

### Double `requestAnimationFrame` after `display` change

When switching an element from `display: none` to `display: block`, the new layout is not measurable until the browser has performed a layout pass. A single `rAF` fires before paint; a **double `rAF`** fires after the first paint, when layout is settled:
```javascript
requestAnimationFrame(function() {
  requestAnimationFrame(function() {
    // DOM is now laid out; scrollIntoView / getBoundingClientRect are accurate
  });
});
```

### `void el.offsetWidth` — force reflow to restart CSS animation

Removing and re-adding a CSS animation class in the same JS task has no effect — the browser batches style updates and sees no change. Reading `offsetWidth` forces an immediate style recalc/reflow, making the removal visible before the re-add:
```javascript
el.classList.remove('pulse');
void el.offsetWidth; // forces reflow
el.classList.add('pulse');
```

### Mobile media-query ordering

Place `@media (max-width: 600px)` blocks **after** the corresponding desktop rules in `style.css`. Same-specificity rules use source order to break ties — putting mobile rules earlier in the file lets desktop rules silently override them on small screens.

### IIFE / `window` exposure for inline handlers

Inline `onclick="myFn()"` attributes resolve against `window`. If a function is scoped inside an IIFE (`(function() { ... })()`), it is **not** on `window` and the handler will throw `ReferenceError`. Either:
- Define the function at top level, or
- Explicitly expose: `window.myFn = myFn;`

---

## Tech Stack (default — adjust as needed)

- **Pure HTML5 / CSS3 / Vanilla JavaScript** — no framework, no build tools unless justified
- **Fonts:** Fontshare (Zodiak) + Google Fonts (Work Sans)
- **Static hosting compatible** — no server required; deployable on Vercel / Netlify / GitHub Pages
- **Responsive:** Mobile-first

If the project needs auth, persistence, or realtime, consider Supabase (Postgres + Realtime + Edge Functions). Anon key is publishable and safe to commit; service-role keys are not.

---

## Git Workflow

Commit and push after every meaningful unit of work. Imperative mood, describe **what and why**. Stage specific files by name (avoid `git add -A` / `git add .` — they pick up `.env` files, large binaries, and editor scratch files).

If deploying via Vercel / Netlify / GitHub Pages: **the host deploys from git** — local file changes do not appear on the live site until committed and pushed to the production branch.

---

## Coding Principles

- Don't add features, refactor, or introduce abstractions beyond what the task requires. Three similar lines is better than a premature abstraction.
- Don't add error handling, fallbacks, or validation for scenarios that can't happen. Only validate at system boundaries (user input, external APIs).
- Default to writing no comments. Only add one when the **why** is non-obvious: a hidden constraint, a subtle invariant, a workaround for a specific bug.
- Don't explain *what* the code does — well-named identifiers already do that. Don't reference the current task or callers ("used by X", "added for the Y flow") — that belongs in the PR description.
- Avoid backwards-compatibility hacks (renaming `_vars`, `// removed` comments, re-exporting deleted types). If something is unused, delete it completely.
- For UI changes, open the page in a browser and exercise the feature before claiming the task is done.

---

## Target Audience

*(Fill in: who visits this site, what they're looking for, what tone and reading level to write for.)*
