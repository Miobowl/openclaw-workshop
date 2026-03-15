# OpenClaw Workshop Interactive Website Design Spec

## Overview

A single-file interactive website replacing PPT for a 20-30 minute workshop introducing OpenClaw and AI agents to a non-technical audience. Covers sections 1 (OpenClaw intro & pitfalls) and 2 (preparation) of the outline; sections 3 (alternatives) and 4 (paper tools) will be live demos.

## Goals

- Replace static slides with an engaging, scrollable interactive experience
- Make OpenClaw concepts accessible to people unfamiliar with AI agents
- Support both presenter-led (projector) and self-browse modes
- Zero build tooling: open the HTML file directly in a browser

## Technical Stack

- **Single HTML file** (HTML + CSS + JS inlined)
- **GSAP ScrollTrigger** (CDN) for scroll-driven animations
- **CSS scroll-snap** for chapter-level page transitions
- **Intersection Observer** for triggering element animations
- No build step, no framework, no dependencies beyond GSAP CDN

## Visual Style

- **Primary gradient:** `#E8520E` to `#FF8C42` (OpenClaw orange)
- **Background:** `#FAFAF8` (warm off-white)
- **Body text:** `#1A1A1A`
- **Secondary text:** `#666666`
- **Code blocks:** `#F5F2EF` background, `#2D2D2D` text
- **Warning cards:** `#E53E3E` (red), `#D69E2E` (yellow), `#3182CE` (blue)
- **Fonts:** `system-ui, -apple-system, sans-serif` for body; `'SF Mono', 'Fira Code', monospace` for code
- Warm, branded feel consistent with OpenClaw's identity

## Navigation

**Hybrid scroll model:** The outer container uses CSS `scroll-snap-type: y mandatory` with each chapter as a snap target (`scroll-snap-align: start; min-height: 100vh`). Chapters whose content exceeds the viewport height are taller than 100vh — the snap container simply snaps to the start of the next chapter when the user scrolls past the current one's content.

- There are no nested scrollable divs. Each chapter is a single tall section in the main scroll flow. Short chapters (e.g., 1.1 Opening) are exactly 100vh. Content-heavy chapters (e.g., 1.4 Workspace) expand to their natural height. The scroll-snap snaps at chapter boundaries.
- **Keyboard:** ArrowDown / Space = scroll within chapter naturally. When reaching the end of a chapter, the next scroll snaps to the next chapter start. No separate "next chapter" shortcut needed.
- Minimal navigation bar fixed at top with chapter indicators (clickable to jump).
- Progress indicator showing current chapter position.
- No mode toggle: the same page works for both presenting (projected, presenter scrolls) and self-browsing (user scrolls at own pace). The content is self-explanatory enough for solo reading; the presenter adds verbal commentary on top.

## Content Structure

### Chapter 1: OpenClaw Introduction & Pitfalls

#### 1.1 Opening ("What is OpenClaw?")

**Content:**
- One-line definition: open-source self-hosted AI Agent platform
- Impact numbers: 280K+ GitHub Stars, 1,075 Contributors, GitHub #1 trending
- Brief positioning vs. existing tools (Manus, Claude Code)

**Interaction:** Numbers animate with a counting-up effect on scroll entry.

#### 1.2 Why OpenClaw Went Viral

**Content:** Three core selling points:
1. **IM Integration** - connects to WhatsApp, Feishu, DingTalk, Slack, etc. via channels
2. **Scheduled & Proactive Tasks** - Heartbeat check-ins, cron-based automation
3. **Agent-like Capabilities** - autonomous tool use, multi-step reasoning, memory

**Interaction:** Three cards flip in sequentially on scroll; each card is clickable to expand with more detail.

#### 1.3 Real-World Use Cases

**Content:**
- Digest: automated daily information aggregation
- Daily Report: auto-generated work summaries

**Interaction:** Scrolling showcase with example screenshots or mockup outputs.

#### 1.4 OpenClaw Workspace (Key Section)

This is the largest and most interactive section, roughly 2-3x the scroll depth of other sections. It uses a persistent two-column layout: file tree on the left (~30% width), detail panel on the right (~70% width).

**Layout:** The file tree stays fixed/sticky on the left as the user scrolls. Clicking a file item updates the right detail panel with a slide-in transition. Only one detail panel is shown at a time. On initial load, the memory files (SOUL/TOOLS/USER) are pre-selected to show the four-layer visualization.

**File tree structure:**
```
openclaw-workspace/
  openclaw.json      -- Main config: model, channels, plugins
  SOUL.md            -- Identity layer: personality, behavior rules
  TOOLS.md           -- Tools layer: tool documentation
  USER.md            -- User layer: preferences, habits, background
  HEARTBEAT.md       -- Heartbeat: scheduled autonomous check-ins
  SKILL.md           -- Skill definitions: triggers, execution logic
  cron/              -- Scheduled task directory
    daily-digest.md
    weekly-report.md
  (session memory)   -- Runtime conversation context (not a file)
```

File tree items use `<button>` elements for keyboard accessibility (Tab to navigate, Enter to select). The currently selected item is highlighted with the orange accent color.

**Detail panels (one per file tree item group):**

**Four-Layer Memory (SOUL.md + TOOLS.md + USER.md + session):**
- CSS-based concentric circle diagram (four nested `div` elements with `border-radius: 50%`, sized proportionally, centered with flexbox)
- Inner core: SOUL.md (`#E8520E`, warmest) -- stable identity, rarely changes
- Second ring: TOOLS.md (`#FF6B35`) -- tool capabilities and documentation
- Third ring: USER.md (`#FF8C42`) -- user-specific preferences, evolves over time
- Outer ring: Session memory (`#FFB380`, lightest) -- temporary, per-conversation context
- Click any ring to highlight it and show its description text below the diagram
- GSAP animates rings expanding outward on first view

**Heartbeat & Cron (HEARTBEAT.md + cron/):**
- HEARTBEAT.md defines when and how the agent proactively reaches out
- cron/ directory holds scheduled task definitions (daily digest, weekly report, etc.)
- CSS timeline visualization: a horizontal line representing 24 hours, with orange dots at trigger points, labeled with task names

**Skills (SKILL.md):**
- SKILL.md defines trigger conditions and execution logic
- Three-tier priority: workspace > user > bundled
- ClawHub marketplace with 13,700+ community skills
- Inline annotated code block showing a sample skill definition

**config.json (openclaw.json):**
- Model selection (which LLM to use)
- Channel configuration (which IM platforms to connect)
- Plugin settings
- Styled as a JSON code block with inline comment annotations highlighted in orange

#### 1.5 Pitfall Guide

**Content:** Three major risks:
1. **Security** (RED) - CVE-2026-25253 RCE vulnerability, ClawHavoc supply chain attacks on ClawHub, fundamental security design flaws
2. **Cost** (YELLOW) - Good models are expensive; cheap models have poor effort-to-value ratio; recommendation to wait 6 months for cheaper models to catch up
3. **Maturity** (BLUE) - Architecture concept is good but implementation is rough; wait for major vendors to build stable versions based on OpenClaw's ideas

**Interaction:** Three color-coded warning cards (`#E53E3E` red / `#D69E2E` yellow / `#3182CE` blue). Click to expand inline (accordion style, same as 1.2 cards -- content expands below the card header, pushing subsequent cards down). Only one card expanded at a time.

### Chapter 2: Getting Prepared

#### 2.1 File over System

**Content:**
- Philosophy: own your data in plain files, not locked in proprietary apps
- Recommend Obsidian for Markdown-based knowledge management
- Why Markdown: portable, version-controllable, AI-readable, future-proof

**Interaction:** Split-screen comparison animation: left side shows data locked in app silos, right side shows free-flowing Markdown files that any tool can read.

#### 2.2 Prompt Methodology

**Content (TBD -- will be filled with specific techniques during implementation):**
- Placeholder section. Concrete prompt techniques and examples will be sourced from tutorials and best practices during the implementation phase.
- Expected format: 3-5 key principles, each with a before/after prompt example shown in code blocks.

**Interaction:** Scrolling content with highlighted key points; code-block style prompt examples.

#### 2.3 Skill Methodology

**Content (TBD -- will be filled with specific techniques during implementation):**
- Placeholder section. Concrete skill design patterns will be sourced from OpenClaw documentation and community examples.
- Expected format: annotated sample SKILL.md showing trigger, description, and execution sections.

**Interaction:** Annotated code block showing a sample SKILL.md with highlighted sections and inline explanatory text.

## Interaction Patterns

All interactive elements use the same two patterns:

### 1. Scroll-Triggered Animations
- Elements fade/slide in as they enter viewport (GSAP ScrollTrigger `once: true`)
- Numbers count up on first view (1.1 stats)
- Diagrams build progressively (1.4 memory circles)

### 2. Inline Accordion Expansion
Used in sections 1.2 (feature cards), 1.4 (file tree detail panels), and 1.5 (warning cards). Behavior:
- Click a card/item header to expand content below it, pushing sibling content down
- Click again or click a different item to collapse it (only one expanded at a time per group)
- CSS `max-height` transition for smooth expand/collapse
- Exception: section 1.4 uses a two-column layout (sticky file tree + detail panel) instead of inline expansion, since it has more content per item

## Layout Structure

```
body (scroll-snap-type: y mandatory)
  [Fixed Nav Bar - chapter dots, clickable labels]
  [Section 1.1 - min-height: 100vh, scroll-snap-align: start]
    [Centered content: title, stats with count-up]
  [Section 1.2 - min-height: 100vh]
    [Three expandable cards in vertical stack]
  [Section 1.3 - min-height: 100vh]
    [Use case showcase]
  [Section 1.4 - natural height (~200vh), scroll-snap-align: start]
    [Two-column: sticky file tree (left 30%) | detail panel (right 70%)]
  [Section 1.5 - min-height: 100vh]
    [Three color-coded accordion cards]
  [Section 2.1 - min-height: 100vh]
    [Split-screen comparison]
  [Section 2.2 - min-height: 100vh]
    [Prompt examples]
  [Section 2.3 - min-height: 100vh]
    [Skill code block]
```

## Responsive Considerations

- Primary target: projector/large screen (1920x1080)
- Secondary: laptop/tablet for self-browsing
- Mobile not a priority but should remain readable
- Font sizes optimized for projection readability

## File Size Budget

- Target: under 200KB total (HTML + inline CSS + inline JS)
- GSAP loaded from CDN as primary, with a `<noscript>` fallback note. If CDN is unreachable, content remains readable (static layout, no animations) since all visuals are CSS/HTML — GSAP only drives animations.
- No images; all visuals built with CSS/SVG/HTML
- Minimal SVG icons where needed

## Out of Scope

- Sections 3 (alternatives: Manus, Workbuddy, Notion AI, Claude Code, Youmind) and 4 (paper tools & prompts) - will be live demos
- Backend or server-side logic
- User authentication or data persistence
- Mobile-first responsive design
- Accessibility beyond basic keyboard navigation (ARIA roles, screen reader support)
- Print styles
