# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Single-file interactive HTML website for a 20-30 minute OpenClaw workshop, replacing PowerPoint slides. Introduces OpenClaw (an open-source AI Agent platform) and AI agents to a non-technical audience. Content is in Chinese (zh-CN).

## Architecture

Everything lives in **`index.html`** (~1300 lines) — a self-contained file with inlined CSS (`<style>`) and JS (`<script>`). No build step, no framework, no package manager.

**External CDN dependencies (loaded in `<head>`):**
- GSAP + ScrollTrigger — scroll-driven animations
- Google Fonts (Bricolage Grotesque, DM Sans, JetBrains Mono)
- Lucide Icons

**Internal structure of index.html:**
1. `<head>` — CDN scripts, `<style>` block with all CSS (CSS custom properties in `:root`)
2. `<body>` — fixed nav bar (`#main-nav`) + 10 `<section>` elements (one per chapter page)
3. `<script>` — all JS at end of body (GSAP animations, accordion logic, file tree interactions, count-up effects, nav highlighting)

## Key Design Patterns

- **CSS scroll-snap** (`scroll-snap-type: y mandatory` on `html`) for chapter-level page transitions. Each `.section` has `scroll-snap-align: start` and `min-height: 100vh`.
- **Scroll-triggered animations** via GSAP ScrollTrigger (`once: true`) — elements fade/slide in on viewport entry.
- **Inline accordion expansion** — click card headers to expand/collapse content via CSS `max-height` transitions. One expanded at a time per group.
- **Section 1.4 (Workspace)** is special: uses a sticky two-column layout (file tree left 30%, detail panel right 70%) instead of accordion.
- **Color system:** Orange gradient (`#E8520E` → `#FF8C42`), warm off-white background (`#FAFAF8`), warning cards use red/yellow/blue.

## Development Workflow

```bash
# "Build" and "run" — just open the file in a browser
open index.html
```

There are no tests, no linter, no build commands. Verification is visual — open in browser and confirm rendering.

## Reference Documents

- **Design spec:** `docs/superpowers/specs/2026-03-15-openclaw-workshop-website-design.md` — full content structure, interaction patterns, visual style definitions
- **Implementation plan:** `docs/superpowers/plans/2026-03-15-openclaw-workshop-website.md` — task breakdown with checkboxes

## Constraints

- Must remain a single HTML file under 200KB
- No images — all visuals built with CSS/SVG/HTML
- Primary target: projector/large screen (1920x1080); secondary: laptop/tablet
- Content is self-explanatory for solo browsing; presenter adds verbal commentary
