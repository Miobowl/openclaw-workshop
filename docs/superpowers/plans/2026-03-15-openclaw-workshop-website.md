# OpenClaw Workshop Website Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file interactive HTML website for a 20-30 minute OpenClaw workshop, replacing PPT slides.

**Architecture:** Single HTML file with inlined CSS and JS. GSAP ScrollTrigger from CDN handles scroll-driven animations. CSS scroll-snap provides chapter-level page transitions. No build step, no framework.

**Tech Stack:** HTML5, CSS3 (scroll-snap, custom properties, flexbox), vanilla JS, GSAP + ScrollTrigger (CDN)

**Spec:** `docs/superpowers/specs/2026-03-15-openclaw-workshop-website-design.md`

**Note:** This is a single-file presentation website, not a software application. There are no unit tests. Verification is visual — open the file in a browser after each task and confirm the section renders correctly.

---

## File Structure

This project produces a single file:

- **Create:** `index.html` — the entire website (HTML structure + `<style>` block + `<script>` block)

The file is organized internally as:
1. `<head>` — meta tags, GSAP CDN script, `<style>` block with all CSS
2. `<body>` — fixed nav bar + 8 `<section>` elements (one per page)
3. `<script>` — all JS at end of body (GSAP animations, accordion logic, file tree interactions, count-up, nav highlighting)

---

## Chunk 1: Scaffold and Chapter 1 Sections

### Task 1: Base Scaffold

**Files:**
- Create: `index.html`

- [ ] **Step 1: Create the HTML file with base structure**

Create `index.html` with:
- `<!DOCTYPE html>`, lang="zh-CN", viewport meta, title "OpenClaw Workshop"
- GSAP + ScrollTrigger CDN links in `<head>` (gsap 3.12.x, ScrollTrigger plugin)
- `<style>` block with CSS custom properties for the color palette:
  - `--orange-start: #E8520E`, `--orange-end: #FF8C42`
  - `--bg: #FAFAF8`, `--text: #1A1A1A`, `--text-secondary: #666666`
  - `--code-bg: #F5F2EF`, `--code-text: #2D2D2D`
  - `--red: #E53E3E`, `--yellow: #D69E2E`, `--blue: #3182CE`
- Base CSS reset (`*, *::before, *::after { margin:0; padding:0; box-sizing: border-box }`)
- `html` with `scroll-snap-type: y mandatory; scroll-behavior: smooth`
- `body` with font-family `system-ui, -apple-system, sans-serif`, color `var(--text)`, background `var(--bg)`
- `.section` class: `min-height: 100vh; scroll-snap-align: start; padding: 80px 10% 60px;` (80px top padding for fixed nav)
- `.section-tall` extends `.section` with `min-height: auto` (overrides `.section`'s `min-height: 100vh` so section 1.4 can grow to its natural content height while keeping `scroll-snap-align: start`)
- `html` also needs `scroll-padding-top: 70px` for proper nav-click scroll offset
- 8 empty `<section>` elements with ids: `s-1-1`, `s-1-2`, `s-1-3`, `s-1-4`, `s-1-5`, `s-2-1`, `s-2-2`, `s-2-3`
- A fixed nav bar at top: `<nav>` with `position: fixed; top: 0; width: 100%; z-index: 100; background: rgba(250,250,248,0.95); backdrop-filter: blur(8px); border-bottom: 1px solid #eee; padding: 12px 10%;`
  - Inside nav: chapter dots/labels as `<a href="#s-1-1">` etc., styled as small circles with labels
- Empty `<script>` block at end of body

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>OpenClaw Workshop</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/ScrollTrigger.min.js"></script>
  <style>
    /* ... CSS as described above ... */
    /* Nav dot styles */
    #main-nav { display: flex; align-items: center; gap: 6px; }
    .nav-dot { display: inline-flex; align-items: center; justify-content: center; width: 32px; height: 32px; border-radius: 50%; background: transparent; color: var(--text-secondary); font-size: 0.7rem; font-weight: 600; text-decoration: none; transition: background 0.3s, color 0.3s; }
    .nav-dot:hover { background: rgba(232,82,14,0.08); }
    .nav-dot.active { background: var(--orange-start); color: white; }
    .nav-sep { color: #ddd; font-size: 0.7rem; margin: 0 4px; } /* separator between chapter groups */
  </style>
</head>
<body>
  <noscript><p style="text-align:center;padding:40px;">This presentation works best with JavaScript enabled for animations. Content is still readable without it.</p></noscript>
  <nav id="main-nav">
    <a href="#s-1-1" class="nav-dot active">1.1</a>
    <a href="#s-1-2" class="nav-dot">1.2</a>
    <a href="#s-1-3" class="nav-dot">1.3</a>
    <a href="#s-1-4" class="nav-dot">1.4</a>
    <a href="#s-1-5" class="nav-dot">1.5</a>
    <span class="nav-sep">|</span>
    <a href="#s-2-1" class="nav-dot">2.1</a>
    <a href="#s-2-2" class="nav-dot">2.2</a>
    <a href="#s-2-3" class="nav-dot">2.3</a>
  </nav>
  <section id="s-1-1" class="section"><!-- 1.1 --></section>
  <section id="s-1-2" class="section"><!-- 1.2 --></section>
  <section id="s-1-3" class="section"><!-- 1.3 --></section>
  <section id="s-1-4" class="section section-tall"><!-- 1.4 --></section>
  <section id="s-1-5" class="section"><!-- 1.5 --></section>
  <section id="s-2-1" class="section"><!-- 2.1 --></section>
  <section id="s-2-2" class="section"><!-- 2.2 --></section>
  <section id="s-2-3" class="section"><!-- 2.3 --></section>
  <script>
    gsap.registerPlugin(ScrollTrigger);
    // Nav active state tracking
  </script>
</body>
</html>
```

- [ ] **Step 2: Add nav bar active-state JS**

In the `<script>` block, use GSAP ScrollTrigger (already loaded) to track which section is in view. This handles both short (100vh) and tall (~200vh) sections correctly, unlike IntersectionObserver with a fixed threshold.

```js
const navDots = document.querySelectorAll('.nav-dot');

document.querySelectorAll('section[id]').forEach(section => {
  ScrollTrigger.create({
    trigger: section,
    start: 'top center',
    end: 'bottom center',
    onToggle: self => {
      if (self.isActive) {
        navDots.forEach(d => d.classList.remove('active'));
        const dot = document.querySelector(`.nav-dot[href="#${section.id}"]`);
        if (dot) dot.classList.add('active');
      }
    }
  });
});
```

- [ ] **Step 3: Verify and commit**

Open `index.html` in browser. Confirm:
- 8 snapping sections visible when scrolling
- Nav bar stays fixed at top
- Nav dots highlight as you scroll between sections

```bash
git add index.html
git commit -m "feat: scaffold HTML with scroll-snap sections and nav bar"
```

---

### Task 2: Section 1.1 — Opening

**Files:**
- Modify: `index.html` (section `#s-1-1` content + JS for count-up)

- [ ] **Step 1: Add HTML content for section 1.1**

Inside `#s-1-1`, add centered layout:

```html
<div class="hero">
  <h1 class="hero-title">
    What is <span class="gradient-text">OpenClaw</span>?
  </h1>
  <p class="hero-subtitle">
    开源、自托管的 AI Agent 平台
  </p>
  <div class="stats">
    <div class="stat">
      <span class="stat-number" data-target="280000">0</span><span class="stat-suffix">+</span>
      <span class="stat-label">GitHub Stars</span>
    </div>
    <div class="stat">
      <span class="stat-number" data-target="1075">0</span>
      <span class="stat-label">Contributors</span>
    </div>
    <div class="stat">
      <span class="stat-number" data-target="1" data-prefix="#">0</span>
      <span class="stat-label">GitHub Trending</span>
    </div>
  </div>
  <p class="hero-note">Manus 和 Claude Code 能做的，OpenClaw 也能做 —— 而且是开源的</p>
</div>
```

- [ ] **Step 2: Add CSS for hero section**

```css
.hero { display: flex; flex-direction: column; align-items: center; justify-content: center; min-height: calc(100vh - 140px); text-align: center; }
.hero-title { font-size: 4rem; font-weight: 800; margin-bottom: 16px; }
.gradient-text { background: linear-gradient(135deg, var(--orange-start), var(--orange-end)); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
.hero-subtitle { font-size: 1.5rem; color: var(--text-secondary); margin-bottom: 48px; }
.stats { display: flex; gap: 64px; margin-bottom: 48px; }
.stat { display: flex; flex-direction: column; align-items: center; }
.stat-number { font-size: 3rem; font-weight: 800; color: var(--orange-start); }
.stat-suffix { font-size: 3rem; font-weight: 800; color: var(--orange-start); }
.stat-label { font-size: 0.9rem; color: var(--text-secondary); margin-top: 4px; }
.hero-note { font-size: 1rem; color: var(--text-secondary); max-width: 600px; }
```

- [ ] **Step 3: Add count-up animation JS**

```js
function animateCountUp(el) {
  const target = parseInt(el.dataset.target);
  const prefix = el.dataset.prefix || '';
  const duration = 2000;
  const start = performance.now();
  function update(now) {
    const elapsed = now - start;
    const progress = Math.min(elapsed / duration, 1);
    const eased = 1 - Math.pow(1 - progress, 3); // ease-out cubic
    const current = Math.floor(eased * target);
    if (target >= 1000) {
      el.textContent = prefix + current.toLocaleString();
    } else {
      el.textContent = prefix + current;
    }
    if (progress < 1) requestAnimationFrame(update);
  }
  requestAnimationFrame(update);
}

ScrollTrigger.create({
  trigger: '#s-1-1',
  start: 'top center',
  once: true,
  onEnter: () => {
    document.querySelectorAll('#s-1-1 .stat-number').forEach(animateCountUp);
  }
});
```

- [ ] **Step 4: Verify and commit**

Open in browser. Confirm: large title with orange gradient text, three stats that count up when scrolled into view.

```bash
git add index.html
git commit -m "feat: add section 1.1 opening with count-up stats"
```

---

### Task 3: Section 1.2 — Why OpenClaw Went Viral

**Files:**
- Modify: `index.html` (section `#s-1-2` + accordion JS)

- [ ] **Step 1: Add HTML for three feature cards**

Inside `#s-1-2`:

```html
<h2 class="section-title">为什么 OpenClaw 会火？</h2>
<div class="card-stack">
  <div class="card" data-card="im">
    <div class="card-header" onclick="toggleCard(this)">
      <span class="card-icon">💬</span>
      <h3>IM 集成</h3>
      <span class="card-arrow">&#9662;</span>
    </div>
    <div class="card-body">
      <p>直接在你常用的聊天工具里使用 AI Agent：WhatsApp、飞书、钉钉、Slack、微信（通过插件）……不需要单独打开一个 App，对话即操作。</p>
    </div>
  </div>
  <div class="card" data-card="schedule">
    <div class="card-header" onclick="toggleCard(this)">
      <span class="card-icon">⏰</span>
      <h3>定时 & 主动任务</h3>
      <span class="card-arrow">&#9662;</span>
    </div>
    <div class="card-body">
      <p>通过 Heartbeat 机制定时"醒来"检查任务，通过 cron 目录定义日报、周报等定时自动执行的任务。不需要你主动发起，Agent 会自己干活。</p>
    </div>
  </div>
  <div class="card" data-card="agent">
    <div class="card-header" onclick="toggleCard(this)">
      <span class="card-icon">🤖</span>
      <h3>类 Agent 能力</h3>
      <span class="card-arrow">&#9662;</span>
    </div>
    <div class="card-body">
      <p>自主调用工具、多步推理、记忆系统 —— 不只是聊天，而是真正能帮你完成任务的智能体。Manus 和 Claude Code 能做的事，OpenClaw 也能做。</p>
    </div>
  </div>
</div>
```

- [ ] **Step 2: Add CSS for cards and accordion**

```css
.section-title { font-size: 2.5rem; font-weight: 800; margin-bottom: 48px; }
.card-stack { max-width: 700px; margin: 0 auto; display: flex; flex-direction: column; gap: 16px; }
.card { background: white; border-radius: 12px; box-shadow: 0 2px 8px rgba(0,0,0,0.06); overflow: hidden; }
.card-header { display: flex; align-items: center; gap: 12px; padding: 20px 24px; cursor: pointer; user-select: none; }
.card-header h3 { flex: 1; font-size: 1.25rem; font-weight: 700; }
.card-icon { font-size: 1.5rem; }
.card-arrow { font-size: 0.8rem; color: var(--text-secondary); transition: transform 0.3s; }
.card.expanded .card-arrow { transform: rotate(180deg); }
.card-body { max-height: 0; overflow: hidden; transition: max-height 0.4s ease, padding 0.4s ease; padding: 0 24px; }
.card.expanded .card-body { max-height: 500px; padding: 0 24px 20px; }
.card-body p { color: var(--text-secondary); line-height: 1.7; }
```

- [ ] **Step 3: Add accordion toggle JS and GSAP entrance animation**

```js
function toggleCard(headerEl) {
  const card = headerEl.parentElement;
  const stack = card.parentElement;
  const wasExpanded = card.classList.contains('expanded');
  stack.querySelectorAll('.card.expanded').forEach(c => c.classList.remove('expanded'));
  if (!wasExpanded) card.classList.add('expanded');
}

// Cards flip in on scroll
gsap.utils.toArray('#s-1-2 .card').forEach((card, i) => {
  gsap.from(card, {
    scrollTrigger: { trigger: card, start: 'top 85%', once: true },
    opacity: 0, y: 40, duration: 0.6, delay: i * 0.15, ease: 'power2.out'
  });
});
```

- [ ] **Step 4: Verify and commit**

Open in browser. Scroll to section 1.2. Confirm: three cards animate in sequentially, clicking a card header expands its body, clicking another collapses the first.

```bash
git add index.html
git commit -m "feat: add section 1.2 with feature cards and accordion"
```

---

### Task 4: Section 1.3 — Real-World Use Cases

**Files:**
- Modify: `index.html` (section `#s-1-3`)

- [ ] **Step 1: Add HTML for use case showcase**

Inside `#s-1-3`:

```html
<h2 class="section-title">我的一些 OpenClaw 用例</h2>
<div class="use-cases">
  <div class="use-case">
    <div class="use-case-mockup digest-mockup">
      <div class="mockup-header">📋 Daily Digest</div>
      <div class="mockup-body">
        <div class="mockup-line"><span class="mockup-tag">科技</span> GPT-5 发布，多模态能力大幅提升</div>
        <div class="mockup-line"><span class="mockup-tag">金融</span> 美联储维持利率不变，市场反应平淡</div>
        <div class="mockup-line"><span class="mockup-tag">开源</span> OpenClaw v2.1 发布，新增插件市场</div>
      </div>
    </div>
    <div class="use-case-text">
      <h3>Digest — 信息聚合</h3>
      <p>每天自动从多个信息源抓取、筛选、分类、总结，生成一份个性化的信息摘要，直接推送到你的 IM 工具。</p>
    </div>
  </div>
  <div class="use-case">
    <div class="use-case-mockup report-mockup">
      <div class="mockup-header">📊 Daily Report</div>
      <div class="mockup-body">
        <div class="mockup-line">✅ 完成：3 个 PR 合并，2 个 Issue 关闭</div>
        <div class="mockup-line">🔄 进行中：API 重构 (60%)</div>
        <div class="mockup-line">📌 明日计划：完成测试覆盖</div>
      </div>
    </div>
    <div class="use-case-text">
      <h3>日报 — 自动生成</h3>
      <p>Agent 自动收集你一天的工作数据（Git 提交、日历、任务管理），整理成结构化的日报，你只需要检查一下就能发出去。</p>
    </div>
  </div>
</div>
```

- [ ] **Step 2: Add CSS for use cases**

```css
.use-cases { max-width: 900px; margin: 0 auto; display: flex; flex-direction: column; gap: 48px; }
.use-case { display: flex; gap: 40px; align-items: center; }
.use-case:nth-child(even) { flex-direction: row-reverse; }
.use-case-mockup { flex: 1; background: white; border-radius: 12px; box-shadow: 0 2px 12px rgba(0,0,0,0.08); overflow: hidden; }
.mockup-header { background: linear-gradient(135deg, var(--orange-start), var(--orange-end)); color: white; padding: 12px 20px; font-weight: 600; }
.mockup-body { padding: 16px 20px; }
.mockup-line { padding: 8px 0; border-bottom: 1px solid #f0f0f0; font-size: 0.9rem; color: var(--text-secondary); }
.mockup-line:last-child { border-bottom: none; }
.mockup-tag { background: rgba(232,82,14,0.1); color: var(--orange-start); padding: 2px 8px; border-radius: 4px; font-size: 0.8rem; margin-right: 8px; }
.use-case-text { flex: 1; }
.use-case-text h3 { font-size: 1.5rem; font-weight: 700; margin-bottom: 12px; }
.use-case-text p { color: var(--text-secondary); line-height: 1.7; }
```

- [ ] **Step 3: Add GSAP scroll animation**

```js
gsap.utils.toArray('.use-case').forEach((uc, i) => {
  gsap.from(uc, {
    scrollTrigger: { trigger: uc, start: 'top 80%', once: true },
    opacity: 0, x: i % 2 === 0 ? -60 : 60, duration: 0.7, ease: 'power2.out'
  });
});
```

- [ ] **Step 4: Verify and commit**

Open in browser. Scroll to 1.3. Confirm: two mockup cards with alternating layout, sliding in from left/right on scroll.

```bash
git add index.html
git commit -m "feat: add section 1.3 use case showcase"
```

---

### Task 5: Section 1.4 — OpenClaw Workspace (Key Section)

**Files:**
- Modify: `index.html` (section `#s-1-4` — this is the largest task)

This task builds the two-column file tree + detail panel layout. It is split into sub-steps: first the layout and file tree, then each detail panel.

- [ ] **Step 1: Add HTML structure for two-column layout and file tree**

Inside `#s-1-4`:

```html
<h2 class="section-title">OpenClaw 工作空间</h2>
<div class="workspace">
  <div class="file-tree">
    <div class="tree-header">📂 openclaw-workspace/</div>
    <button class="tree-item" data-panel="config" onclick="showPanel(this)">
      <span class="tree-indent">  </span>📄 openclaw.json
    </button>
    <button class="tree-item active" data-panel="memory" onclick="showPanel(this)">
      <span class="tree-indent">  </span>📄 SOUL.md
    </button>
    <button class="tree-item active" data-panel="memory" onclick="showPanel(this)">
      <span class="tree-indent">  </span>📄 TOOLS.md
    </button>
    <button class="tree-item active" data-panel="memory" onclick="showPanel(this)">
      <span class="tree-indent">  </span>📄 USER.md
    </button>
    <button class="tree-item" data-panel="heartbeat" onclick="showPanel(this)">
      <span class="tree-indent">  </span>📄 HEARTBEAT.md
    </button>
    <button class="tree-item" data-panel="skill" onclick="showPanel(this)">
      <span class="tree-indent">  </span>📄 SKILL.md
    </button>
    <button class="tree-item" data-panel="heartbeat" onclick="showPanel(this)">
      <span class="tree-indent">  </span>📂 cron/
    </button>
    <div class="tree-sub">
      <button class="tree-item" data-panel="heartbeat" onclick="showPanel(this)">
        <span class="tree-indent">    </span>📄 daily-digest.md
      </button>
      <button class="tree-item" data-panel="heartbeat" onclick="showPanel(this)">
        <span class="tree-indent">    </span>📄 weekly-report.md
      </button>
    </div>
    <div class="tree-session">
      <span class="tree-indent">  </span>💭 (session memory)
    </div>
  </div>
  <div class="detail-area">
    <div class="detail-panel active" id="panel-memory"><!-- filled in step 2 --></div>
    <div class="detail-panel" id="panel-heartbeat"><!-- filled in step 3 --></div>
    <div class="detail-panel" id="panel-skill"><!-- filled in step 4 --></div>
    <div class="detail-panel" id="panel-config"><!-- filled in step 5 --></div>
  </div>
</div>
```

Add CSS:

```css
.workspace { display: flex; gap: 32px; min-height: 70vh; }
.file-tree { width: 30%; position: sticky; top: 100px; align-self: flex-start; background: white; border-radius: 12px; box-shadow: 0 2px 8px rgba(0,0,0,0.06); padding: 20px; }
.tree-header { font-weight: 700; font-size: 1.1rem; margin-bottom: 12px; padding-bottom: 12px; border-bottom: 1px solid #eee; font-family: 'SF Mono', 'Fira Code', monospace; }
.tree-item { display: block; width: 100%; text-align: left; background: none; border: none; font-family: 'SF Mono', 'Fira Code', monospace; font-size: 0.85rem; padding: 10px 12px; border-radius: 8px; cursor: pointer; color: var(--text); transition: background 0.2s; }
.tree-item:hover { background: #f5f5f5; }
.tree-item.active { background: rgba(232,82,14,0.1); color: var(--orange-start); font-weight: 600; }
.tree-indent { white-space: pre; }
.tree-sub { margin-left: 0; }
.tree-session { padding: 10px 12px; font-size: 0.85rem; color: var(--text-secondary); font-family: 'SF Mono', 'Fira Code', monospace; }
.detail-area { width: 70%; }
.detail-panel { display: none; animation: fadeSlideIn 0.3s ease; }
.detail-panel.active { display: block; }
@keyframes fadeSlideIn { from { opacity: 0; transform: translateX(20px); } to { opacity: 1; transform: translateX(0); } }
```

Add JS:

```js
function showPanel(btn) {
  const panelName = btn.dataset.panel;
  // Highlight all tree items that map to this panel
  document.querySelectorAll('.tree-item').forEach(b => {
    b.classList.toggle('active', b.dataset.panel === panelName);
  });
  const panelId = 'panel-' + panelName;
  document.querySelectorAll('.detail-panel').forEach(p => {
    p.classList.remove('active');
    p.style.animation = 'none';
  });
  const panel = document.getElementById(panelId);
  panel.classList.add('active');
  // Re-trigger animation
  panel.offsetHeight;
  panel.style.animation = 'fadeSlideIn 0.3s ease';
}
```

- [ ] **Step 2: Add memory panel with concentric circles diagram**

Inside `#panel-memory`:

```html
<h3 class="panel-title">四层记忆系统</h3>
<p class="panel-desc">OpenClaw 用四层文件来定义 Agent 的"记忆"，从内到外，从稳定到临时：</p>
<div class="memory-diagram">
  <div class="memory-ring ring-session" data-layer="session" onclick="highlightRing(this)">
    <div class="memory-ring ring-user" data-layer="user" onclick="highlightRing(this); event.stopPropagation();">
      <div class="memory-ring ring-tools" data-layer="tools" onclick="highlightRing(this); event.stopPropagation();">
        <div class="memory-ring ring-soul" data-layer="soul" onclick="highlightRing(this); event.stopPropagation();">
          SOUL
        </div>
      </div>
    </div>
  </div>
</div>
<div class="memory-labels">
  <div class="memory-label" data-for="soul"><strong>SOUL.md</strong> — 身份与人格：我是谁？我的行为准则是什么？几乎不变。</div>
  <div class="memory-label" data-for="tools"><strong>TOOLS.md</strong> — 工具文档：我能用什么工具？怎么用？</div>
  <div class="memory-label" data-for="user"><strong>USER.md</strong> — 用户偏好：用户喜欢什么风格？有什么习惯？逐渐积累。</div>
  <div class="memory-label active" data-for="session"><strong>Session</strong> — 会话记忆：当前对话的上下文，对话结束即清除。</div>
</div>
```

Add CSS for the concentric circles:

```css
.panel-title { font-size: 1.5rem; font-weight: 700; margin-bottom: 8px; }
.panel-desc { color: var(--text-secondary); margin-bottom: 32px; line-height: 1.6; }
.memory-diagram { display: flex; align-items: center; justify-content: center; margin-bottom: 32px; }
.memory-ring { border-radius: 50%; display: flex; align-items: center; justify-content: center; cursor: pointer; transition: transform 0.3s, box-shadow 0.3s; }
.ring-session { width: 320px; height: 320px; background: #FFB380; }
.ring-user { width: 240px; height: 240px; background: #FF8C42; }
.ring-tools { width: 160px; height: 160px; background: #FF6B35; }
.ring-soul { width: 90px; height: 90px; background: var(--orange-start); color: white; font-weight: 700; font-size: 0.85rem; }
.memory-ring:hover { transform: scale(1.05); box-shadow: 0 0 0 3px rgba(232,82,14,0.3); }
.memory-ring.highlighted { box-shadow: 0 0 0 4px var(--orange-start); transform: scale(1.05); }
.memory-labels { display: flex; flex-direction: column; gap: 8px; }
.memory-label { padding: 12px 16px; border-radius: 8px; background: #f9f9f7; color: var(--text-secondary); line-height: 1.5; transition: background 0.3s; }
.memory-label.active { background: rgba(232,82,14,0.08); color: var(--text); }
```

Add JS:

```js
function highlightRing(el) {
  document.querySelectorAll('.memory-ring').forEach(r => r.classList.remove('highlighted'));
  el.classList.add('highlighted');
  const layer = el.dataset.layer;
  document.querySelectorAll('.memory-label').forEach(l => l.classList.remove('active'));
  document.querySelector(`.memory-label[data-for="${layer}"]`).classList.add('active');
}

// GSAP entrance: rings appear outside-in (parent first so children are visible)
ScrollTrigger.create({
  trigger: '#s-1-4',
  start: 'top 60%',
  once: true,
  onEnter: () => {
    gsap.from('.ring-session', { opacity: 0, scale: 0.8, duration: 0.4, ease: 'back.out(1.7)' });
    gsap.from('.ring-user', { opacity: 0, scale: 0.8, duration: 0.4, delay: 0.15, ease: 'back.out(1.7)' });
    gsap.from('.ring-tools', { opacity: 0, scale: 0.8, duration: 0.4, delay: 0.3, ease: 'back.out(1.7)' });
    gsap.from('.ring-soul', { opacity: 0, scale: 0.8, duration: 0.4, delay: 0.45, ease: 'back.out(1.7)' });
  }
});
```

- [ ] **Step 3: Add heartbeat & cron panel with timeline**

Inside `#panel-heartbeat`:

```html
<h3 class="panel-title">Heartbeat & Cron</h3>
<p class="panel-desc">OpenClaw 不只是被动回答问题。它能主动"醒来"执行任务。</p>
<div class="subsection">
  <h4>HEARTBEAT.md — 主动检查</h4>
  <p>定义 Agent 什么时候主动联系你。比如每 4 小时检查一次待办事项，有紧急邮件时立即通知。</p>
</div>
<div class="subsection">
  <h4>cron/ 目录 — 定时任务</h4>
  <p>像 Linux 的 crontab 一样，用 Markdown 文件定义定时任务：</p>
</div>
<div class="timeline">
  <div class="timeline-line"></div>
  <div class="timeline-point" style="left: 8%"><span class="tp-label">6:00<br>晨报</span></div>
  <div class="timeline-point" style="left: 25%"><span class="tp-label">9:00<br>邮件检查</span></div>
  <div class="timeline-point" style="left: 42%"><span class="tp-label">12:00<br>午间摘要</span></div>
  <div class="timeline-point" style="left: 58%"><span class="tp-label">15:00<br>进度检查</span></div>
  <div class="timeline-point" style="left: 75%"><span class="tp-label">18:00<br>日报生成</span></div>
  <div class="timeline-point" style="left: 92%"><span class="tp-label">22:00<br>明日计划</span></div>
</div>
```

Add CSS:

```css
.subsection { margin-bottom: 20px; }
.subsection h4 { font-size: 1.1rem; font-weight: 700; margin-bottom: 6px; }
.subsection p { color: var(--text-secondary); line-height: 1.6; }
.timeline { position: relative; height: 100px; margin: 32px 0; }
.timeline-line { position: absolute; top: 50%; left: 0; right: 0; height: 3px; background: linear-gradient(90deg, var(--orange-start), var(--orange-end)); border-radius: 2px; }
.timeline-point { position: absolute; top: 50%; transform: translate(-50%, -50%); width: 16px; height: 16px; background: var(--orange-start); border-radius: 50%; border: 3px solid white; box-shadow: 0 0 0 2px var(--orange-start); }
.tp-label { position: absolute; top: 24px; left: 50%; transform: translateX(-50%); white-space: nowrap; font-size: 0.75rem; color: var(--text-secondary); text-align: center; }
```

- [ ] **Step 4: Add skills panel**

Inside `#panel-skill`:

```html
<h3 class="panel-title">Skills 技能系统</h3>
<p class="panel-desc">Skills 是 OpenClaw 的能力扩展机制 —— 通过 Markdown 文件定义 Agent 能做什么、什么时候触发。</p>
<div class="subsection">
  <h4>三层优先级</h4>
  <div class="priority-stack">
    <div class="priority-item p-high">🏠 Workspace 级别<span class="priority-note">当前项目专属，最高优先</span></div>
    <div class="priority-item p-mid">👤 User 级别<span class="priority-note">用户自定义，跨项目通用</span></div>
    <div class="priority-item p-low">📦 Bundled 级别<span class="priority-note">内置技能，兜底</span></div>
  </div>
</div>
<div class="subsection">
  <h4>ClawHub 技能市场</h4>
  <p>社区贡献了 <strong>13,700+</strong> 技能，一键安装，涵盖开发、写作、数据分析等各个领域。</p>
</div>
<div class="subsection">
  <h4>Skill 示例</h4>
  <pre class="code-block"><code><span class="code-comment"># daily-digest.md</span>
<span class="code-key">name:</span> daily-digest
<span class="code-key">trigger:</span> cron("0 8 * * *")  <span class="code-comment"># 每天早上 8 点</span>
<span class="code-key">description:</span> |
  从 RSS、邮件、GitHub 收集信息，
  生成个性化摘要，发送到飞书。</code></pre>
</div>
```

Add CSS:

```css
.priority-stack { display: flex; flex-direction: column; gap: 8px; margin: 12px 0; }
.priority-item { padding: 12px 16px; border-radius: 8px; font-weight: 600; display: flex; justify-content: space-between; align-items: center; }
.priority-note { font-weight: 400; font-size: 0.85rem; color: var(--text-secondary); }
.p-high { background: rgba(232,82,14,0.12); color: var(--orange-start); }
.p-mid { background: rgba(232,82,14,0.06); color: #cc6a20; }
.p-low { background: #f5f5f3; color: var(--text-secondary); }
.code-block { background: var(--code-bg); color: var(--code-text); padding: 20px; border-radius: 10px; font-family: 'SF Mono', 'Fira Code', monospace; font-size: 0.85rem; line-height: 1.7; overflow-x: auto; }
.code-comment { color: #999; }
.code-key { color: var(--orange-start); font-weight: 600; }
```

- [ ] **Step 5: Add config panel**

Inside `#panel-config`:

```html
<h3 class="panel-title">openclaw.json 配置文件</h3>
<p class="panel-desc">OpenClaw 的主控配置，决定了 Agent 用什么模型、连什么平台、启用什么插件。</p>
<pre class="code-block"><code>{
  <span class="code-key">"model"</span>: {
    <span class="code-key">"name"</span>: <span class="code-str">"claude-sonnet-4-6"</span>,     <span class="code-comment">// 使用的大模型</span>
    <span class="code-key">"maxTokens"</span>: <span class="code-num">8192</span>              <span class="code-comment">// 单次最大输出</span>
  },
  <span class="code-key">"channels"</span>: [
    {
      <span class="code-key">"type"</span>: <span class="code-str">"feishu"</span>,              <span class="code-comment">// 飞书通道</span>
      <span class="code-key">"appId"</span>: <span class="code-str">"cli_xxx"</span>,
      <span class="code-key">"appSecret"</span>: <span class="code-str">"${FEISHU_SECRET}"</span> <span class="code-comment">// 环境变量引用</span>
    },
    {
      <span class="code-key">"type"</span>: <span class="code-str">"dingtalk"</span>,            <span class="code-comment">// 钉钉通道</span>
      <span class="code-key">"webhook"</span>: <span class="code-str">"https://..."</span>
    }
  ],
  <span class="code-key">"plugins"</span>: [<span class="code-str">"web-search"</span>, <span class="code-str">"code-runner"</span>]  <span class="code-comment">// 启用的插件</span>
}</code></pre>
```

Add CSS:

```css
.code-str { color: #22863a; }
.code-num { color: #005cc5; }
```

- [ ] **Step 6: Verify and commit**

Open in browser. Scroll to 1.4. Confirm:
- File tree is sticky on left, detail panel switches on right
- Memory circles animate in on first view, clicking a ring highlights it and shows description
- Timeline shows dots along a 24h line
- Skill and config panels show formatted code blocks

```bash
git add index.html
git commit -m "feat: add section 1.4 workspace with file tree and detail panels"
```

---

### Task 6: Section 1.5 — Pitfall Guide

**Files:**
- Modify: `index.html` (section `#s-1-5`)

- [ ] **Step 1: Add HTML for three warning cards**

Inside `#s-1-5`:

```html
<h2 class="section-title">避坑指南</h2>
<p class="section-subtitle">OpenClaw 很酷，但现在入坑要三思。</p>
<div class="warning-stack">
  <div class="warning-card warning-red">
    <div class="warning-header" onclick="toggleWarning(this.parentElement)">
      <span class="warning-badge badge-red">安全风险</span>
      <h3>安全，安全，还是安全</h3>
      <span class="card-arrow">&#9662;</span>
    </div>
    <div class="warning-body">
      <ul>
        <li><strong>CVE-2026-25253</strong>：远程代码执行漏洞，攻击者可通过恶意 Skill 执行任意命令</li>
        <li><strong>ClawHavoc 供应链攻击</strong>：ClawHub 上的恶意技能伪装成热门工具，窃取用户数据</li>
        <li>安全设计存在根本性缺陷，社区已有多起安全事故</li>
      </ul>
    </div>
  </div>
  <div class="warning-card warning-yellow">
    <div class="warning-header" onclick="toggleWarning(this.parentElement)">
      <span class="warning-badge badge-yellow">成本问题</span>
      <h3>好模型贵，便宜模型性价比低</h3>
      <span class="card-arrow">&#9662;</span>
    </div>
    <div class="warning-body">
      <ul>
        <li>效果好的模型（Claude、GPT-4）费用高昂，日常使用成本不低</li>
        <li>便宜模型需要花大量时间调教和修复，<strong>精力性价比</strong>很低</li>
        <li>建议：等半年，国内便宜模型追上当前第一梯队的 80% 再尝试</li>
      </ul>
    </div>
  </div>
  <div class="warning-card warning-blue">
    <div class="warning-header" onclick="toggleWarning(this.parentElement)">
      <span class="warning-badge badge-blue">成熟度</span>
      <h3>思路好，但实现粗糙</h3>
      <span class="card-arrow">&#9662;</span>
    </div>
    <div class="warning-body">
      <ul>
        <li>OpenClaw 的架构设计理念确实领先，但代码实现质量参差不齐</li>
        <li>当前各大厂出的都只是 OpenClaw 的套壳版本</li>
        <li>建议：等半年，让大厂根据 OpenClaw 的思路开发出更稳定、易用的版本</li>
      </ul>
    </div>
  </div>
</div>
```

- [ ] **Step 2: Add CSS for warning cards**

```css
.section-subtitle { color: var(--text-secondary); font-size: 1.1rem; margin-top: -36px; margin-bottom: 48px; text-align: center; }
.warning-stack { max-width: 700px; margin: 0 auto; display: flex; flex-direction: column; gap: 16px; }
.warning-card { border-radius: 12px; overflow: hidden; transition: box-shadow 0.3s; }
.warning-card:hover { box-shadow: 0 4px 16px rgba(0,0,0,0.1); }
.warning-red { background: #FFF5F5; border: 1px solid #FED7D7; }
.warning-yellow { background: #FFFFF0; border: 1px solid #FEFCBF; }
.warning-blue { background: #EBF8FF; border: 1px solid #BEE3F8; }
.warning-header { display: flex; align-items: center; gap: 12px; padding: 20px 24px; cursor: pointer; user-select: none; }
.warning-header h3 { flex: 1; font-size: 1.1rem; }
.warning-badge { padding: 4px 12px; border-radius: 20px; font-size: 0.8rem; font-weight: 600; }
.badge-red { background: var(--red); color: white; }
.badge-yellow { background: var(--yellow); color: white; }
.badge-blue { background: var(--blue); color: white; }
.warning-body { max-height: 0; overflow: hidden; transition: max-height 0.4s ease, padding 0.4s ease; padding: 0 24px; }
.warning-card.expanded .warning-body { max-height: 500px; padding: 0 24px 20px; }
.warning-card.expanded .card-arrow { transform: rotate(180deg); }
.warning-body ul { list-style: none; padding: 0; }
.warning-body li { padding: 6px 0; color: var(--text-secondary); line-height: 1.6; padding-left: 20px; position: relative; }
.warning-body li::before { content: "⚠"; position: absolute; left: 0; }
```

- [ ] **Step 3: Add toggle JS and entrance animation**

```js
function toggleWarning(card) {
  const stack = card.parentElement;
  const wasExpanded = card.classList.contains('expanded');
  stack.querySelectorAll('.warning-card.expanded').forEach(c => c.classList.remove('expanded'));
  if (!wasExpanded) card.classList.add('expanded');
}

gsap.utils.toArray('.warning-card').forEach((card, i) => {
  gsap.from(card, {
    scrollTrigger: { trigger: card, start: 'top 85%', once: true },
    opacity: 0, y: 30, duration: 0.5, delay: i * 0.12, ease: 'power2.out'
  });
});
```

- [ ] **Step 4: Verify and commit**

Open in browser. Scroll to 1.5. Confirm: three color-coded cards animate in, click to expand, accordion behavior (one at a time).

```bash
git add index.html
git commit -m "feat: add section 1.5 pitfall guide with warning cards"
```

---

## Chunk 2: Chapter 2 Sections and Polish

### Task 7: Section 2.1 — File over System

**Files:**
- Modify: `index.html` (section `#s-2-1`)

- [ ] **Step 1: Add HTML for split-screen comparison**

Inside `#s-2-1`:

```html
<h2 class="section-title">File over System</h2>
<p class="section-subtitle">在 Agent 时代到来之前，先构建好你的知识基础设施。</p>
<div class="comparison">
  <div class="compare-side compare-bad">
    <div class="compare-label">传统方式 ❌</div>
    <div class="compare-box">
      <div class="silo">📱 Notion<div class="silo-lock">🔒 数据锁死</div></div>
      <div class="silo">📱 Evernote<div class="silo-lock">🔒 格式私有</div></div>
      <div class="silo">📱 Roam<div class="silo-lock">🔒 导出困难</div></div>
    </div>
    <p>数据分散在各个 App 里，格式不通用，AI 读不了。</p>
  </div>
  <div class="compare-arrow">→</div>
  <div class="compare-side compare-good">
    <div class="compare-label">File over System ✅</div>
    <div class="compare-box">
      <div class="file-icon">📄 knowledge.md</div>
      <div class="file-icon">📄 projects.md</div>
      <div class="file-icon">📄 journal.md</div>
      <div class="file-arrows">↕ AI 可读 ↕ Git 可控 ↕ 任意工具可编辑</div>
    </div>
    <p>Markdown 文件，本地存储，任何工具都能读写，天然适配 AI Agent。推荐 <strong>Obsidian</strong>。</p>
  </div>
</div>
```

- [ ] **Step 2: Add CSS**

```css
.comparison { display: flex; align-items: stretch; gap: 24px; max-width: 900px; margin: 0 auto; }
.compare-side { flex: 1; padding: 32px; border-radius: 12px; }
.compare-bad { background: #FFF5F5; border: 1px solid #FED7D7; }
.compare-good { background: #F0FFF4; border: 1px solid #C6F6D5; }
.compare-label { font-weight: 700; font-size: 1.1rem; margin-bottom: 20px; }
.compare-box { display: flex; flex-direction: column; gap: 12px; margin-bottom: 20px; }
.silo { background: white; padding: 12px 16px; border-radius: 8px; display: flex; justify-content: space-between; align-items: center; font-weight: 600; }
.silo-lock { font-size: 0.8rem; color: var(--red); }
.file-icon { background: white; padding: 12px 16px; border-radius: 8px; font-family: 'SF Mono', 'Fira Code', monospace; font-size: 0.9rem; }
.file-arrows { text-align: center; font-size: 0.85rem; color: #38A169; font-weight: 600; }
.compare-side p { color: var(--text-secondary); line-height: 1.6; }
.compare-arrow { display: flex; align-items: center; font-size: 2rem; color: var(--text-secondary); }
```

- [ ] **Step 3: Add GSAP animation**

```js
gsap.from('.compare-bad', {
  scrollTrigger: { trigger: '.comparison', start: 'top 75%', once: true },
  opacity: 0, x: -50, duration: 0.6, ease: 'power2.out'
});
gsap.from('.compare-good', {
  scrollTrigger: { trigger: '.comparison', start: 'top 75%', once: true },
  opacity: 0, x: 50, duration: 0.6, delay: 0.2, ease: 'power2.out'
});
```

- [ ] **Step 4: Verify and commit**

Open in browser. Scroll to 2.1. Confirm: split comparison slides in from both sides.

```bash
git add index.html
git commit -m "feat: add section 2.1 file over system comparison"
```

---

### Task 8: Sections 2.2 & 2.3 — Prompt and Skill Methodology

**Files:**
- Modify: `index.html` (sections `#s-2-2` and `#s-2-3`)

- [ ] **Step 1: Add HTML for section 2.2 Prompt Methodology**

Inside `#s-2-2`:

```html
<h2 class="section-title">Prompt 方法论</h2>
<p class="section-subtitle">写好 Prompt 是用好 AI 的基础。</p>
<div class="principles">
  <div class="principle">
    <span class="principle-num">01</span>
    <div class="principle-content">
      <h3>明确角色和目标</h3>
      <div class="prompt-compare">
        <div class="prompt-example bad-prompt">
          <div class="prompt-label">❌ 模糊</div>
          <code>帮我写个总结</code>
        </div>
        <div class="prompt-example good-prompt">
          <div class="prompt-label">✅ 清晰</div>
          <code>你是一位技术博客编辑。请将以下会议纪要总结成 3 个要点，每个不超过 50 字，面向开发者读者。</code>
        </div>
      </div>
    </div>
  </div>
  <div class="principle">
    <span class="principle-num">02</span>
    <div class="principle-content">
      <h3>提供充分的上下文</h3>
      <div class="prompt-compare">
        <div class="prompt-example bad-prompt">
          <div class="prompt-label">❌ 缺少上下文</div>
          <code>这个 bug 怎么修？</code>
        </div>
        <div class="prompt-example good-prompt">
          <div class="prompt-label">✅ 上下文充分</div>
          <code>我在用 Python 3.11 + FastAPI。调用 /api/users 时返回 422 错误。请求体是 {"name": "test"}，模型定义如下：[代码]。帮我找出原因。</code>
        </div>
      </div>
    </div>
  </div>
  <div class="principle">
    <span class="principle-num">03</span>
    <div class="principle-content">
      <h3>指定输出格式</h3>
      <div class="prompt-compare">
        <div class="prompt-example bad-prompt">
          <div class="prompt-label">❌ 无格式要求</div>
          <code>分析这些数据</code>
        </div>
        <div class="prompt-example good-prompt">
          <div class="prompt-label">✅ 格式明确</div>
          <code>请以 Markdown 表格输出，列包括：指标名称、本周数值、环比变化、趋势判断（上升/下降/持平）。</code>
        </div>
      </div>
    </div>
  </div>
</div>
```

- [ ] **Step 2: Add CSS for prompt methodology**

```css
.principles { max-width: 750px; margin: 0 auto; display: flex; flex-direction: column; gap: 40px; }
.principle { display: flex; gap: 20px; }
.principle-num { font-size: 2rem; font-weight: 800; color: var(--orange-start); opacity: 0.3; min-width: 50px; }
.principle-content { flex: 1; }
.principle-content h3 { font-size: 1.25rem; margin-bottom: 16px; }
.prompt-compare { display: flex; flex-direction: column; gap: 10px; }
.prompt-example { padding: 14px 18px; border-radius: 8px; font-family: 'SF Mono', 'Fira Code', monospace; font-size: 0.85rem; line-height: 1.6; }
.bad-prompt { background: #FFF5F5; border-left: 3px solid var(--red); }
.good-prompt { background: #F0FFF4; border-left: 3px solid #38A169; }
.prompt-label { font-family: system-ui, sans-serif; font-size: 0.75rem; font-weight: 600; margin-bottom: 6px; }
.prompt-example code { white-space: pre-wrap; color: var(--text); }
```

- [ ] **Step 3: Add HTML for section 2.3 Skill Methodology**

Inside `#s-2-3`:

```html
<h2 class="section-title">Skill 方法论</h2>
<p class="section-subtitle">学会写 Skill，就是学会教 Agent 做事。</p>
<div class="skill-anatomy">
  <pre class="code-block code-annotated"><code><span class="code-comment"># 一个完整的 Skill 文件结构</span>

<span class="code-section" data-annotation="触发条件：什么时候执行这个 Skill？">---
<span class="code-key">name:</span> weekly-report
<span class="code-key">trigger:</span> cron("0 17 * * 5")  <span class="code-comment"># 每周五 17:00</span>
<span class="code-key">description:</span> 生成本周工作周报
---</span>

<span class="code-section" data-annotation="执行指令：告诉 Agent 具体怎么做。">## 任务

1. 收集本周的 Git commit 记录
2. 汇总已完成和进行中的任务
3. 整理成周报格式，包含：
   - 本周完成的工作
   - 遇到的问题
   - 下周计划</span>

<span class="code-section" data-annotation="输出格式：定义最终产出的样子。">## 输出要求

- 格式：Markdown
- 长度：300-500 字
- 发送到：飞书群 "技术周报"</span></code></pre>
  <div class="annotations">
    <div class="annotation" data-for="0">
      <span class="annotation-marker">1</span>
      <strong>触发条件</strong> — 定义 Skill 的执行时机。支持 cron 表达式、关键词触发、事件触发等。
    </div>
    <div class="annotation" data-for="1">
      <span class="annotation-marker">2</span>
      <strong>执行指令</strong> — Skill 的核心。清晰地告诉 Agent 要做什么，步骤越具体越好。
    </div>
    <div class="annotation" data-for="2">
      <span class="annotation-marker">3</span>
      <strong>输出要求</strong> — 约束最终产出的格式、长度和去向，避免 Agent 自由发挥。
    </div>
  </div>
</div>
```

- [ ] **Step 4: Add CSS for skill anatomy**

```css
.skill-anatomy { max-width: 800px; margin: 0 auto; display: flex; gap: 32px; }
.code-annotated { flex: 1.2; }
.code-section { display: block; padding: 4px 0; border-left: 3px solid transparent; padding-left: 8px; margin-left: -11px; /* -(border-left 3px + padding-left 8px) to align with parent */ transition: border-color 0.3s, background 0.3s; }
.code-section:hover { border-left-color: var(--orange-start); background: rgba(232,82,14,0.04); }
.annotations { flex: 0.8; display: flex; flex-direction: column; gap: 16px; padding-top: 20px; }
.annotation { padding: 16px; background: white; border-radius: 10px; box-shadow: 0 2px 8px rgba(0,0,0,0.06); font-size: 0.9rem; line-height: 1.6; color: var(--text-secondary); transition: opacity 0.3s, transform 0.3s; }
.annotation-marker { display: inline-flex; align-items: center; justify-content: center; width: 24px; height: 24px; background: var(--orange-start); color: white; border-radius: 50%; font-size: 0.75rem; font-weight: 700; margin-right: 8px; }
```

- [ ] **Step 5: Add GSAP animations for both sections**

```js
gsap.utils.toArray('.principle').forEach((p, i) => {
  gsap.from(p, {
    scrollTrigger: { trigger: p, start: 'top 85%', once: true },
    opacity: 0, y: 30, duration: 0.5, delay: i * 0.1, ease: 'power2.out'
  });
});

gsap.from('.code-annotated', {
  scrollTrigger: { trigger: '.skill-anatomy', start: 'top 75%', once: true },
  opacity: 0, x: -40, duration: 0.6, ease: 'power2.out'
});
gsap.from('.annotations', {
  scrollTrigger: { trigger: '.skill-anatomy', start: 'top 75%', once: true },
  opacity: 0, x: 40, duration: 0.6, delay: 0.2, ease: 'power2.out'
});

// Link code sections to annotations on hover
document.querySelectorAll('.code-section').forEach((section, i) => {
  const annotations = document.querySelectorAll('.annotation');
  section.addEventListener('mouseenter', () => {
    annotations.forEach(a => a.style.opacity = '0.4');
    if (annotations[i]) { annotations[i].style.opacity = '1'; annotations[i].style.transform = 'scale(1.02)'; }
  });
  section.addEventListener('mouseleave', () => {
    annotations.forEach(a => { a.style.opacity = '1'; a.style.transform = 'scale(1)'; });
  });
});
```

- [ ] **Step 6: Verify and commit**

Open in browser. Scroll to 2.2 and 2.3. Confirm: prompt principles with before/after examples animate in. Skill anatomy shows code + annotations side by side.

```bash
git add index.html
git commit -m "feat: add sections 2.2 prompt and 2.3 skill methodology"
```

---

### Task 9: Final Polish

**Files:**
- Modify: `index.html` (CSS and JS refinements)

- [ ] **Step 1: Add responsive adjustments for projection**

```css
@media (min-width: 1400px) {
  .hero-title { font-size: 5rem; }
  .stat-number { font-size: 4rem; }
  .section-title { font-size: 3rem; }
}

@media (max-width: 768px) {
  .section { padding: 80px 5% 40px; }
  .hero-title { font-size: 2.5rem; }
  .stats { flex-direction: column; gap: 24px; }
  .comparison { flex-direction: column; }
  .compare-arrow { transform: rotate(90deg); justify-content: center; }
  .workspace { flex-direction: column; }
  .file-tree { width: 100%; position: static; }
  .detail-area { width: 100%; }
  .use-case, .use-case:nth-child(even) { flex-direction: column; }
  .skill-anatomy { flex-direction: column; }
}
```

- [ ] **Step 2: Add text centering and scroll offset**

Add `text-align: center` to `.section-title` for consistent centering across all sections. Add `scroll-padding-top` to `html` for proper nav-click scroll offset:

```css
.section-title { text-align: center; }
html { scroll-padding-top: 70px; }
```

- [ ] **Step 3: Verify full website end-to-end**

Open `index.html` in browser. Walk through all 8 sections:
- [ ] 1.1: Title renders, stats count up
- [ ] 1.2: Three cards animate in, accordion works
- [ ] 1.3: Use cases slide in from alternating sides
- [ ] 1.4: File tree sticky, panels switch, memory circles animate, timeline renders
- [ ] 1.5: Warning cards with color coding, accordion works
- [ ] 2.1: Split comparison slides in
- [ ] 2.2: Prompt principles animate in with before/after examples
- [ ] 2.3: Skill code + annotations side by side
- [ ] Nav bar: dots highlight correctly per section
- [ ] Scroll-snap: snaps at chapter boundaries

- [ ] **Step 4: Commit final polish**

```bash
git add index.html
git commit -m "feat: add responsive styles and final animation polish"
```
