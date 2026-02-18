# OpenClaw UI — Analysis & Action Plan
**Analyzed:** Feb 18, 2026

---

## 📊 What You Have

### 1. Static Mockup (`openclaw-ui-mockup.html`) ✅
- **Quality:** Excellent — production-grade design
- **Layout:** 3-panel (Sidebar + Chat + Dashboard)
- **Features shown:** Thinking blocks, subagent cards, tool calls, code diffs, approval bar, task planner
- **Tech:** Pure HTML/CSS, single file, ~900 lines
- **Status:** Static demo only — no real data, no interactivity

### 2. Full Feature Plan (`openclaw-ui-plan.md`) ✅
- **30 features** across 3 priority tiers
- **P0 (Must Have):** 7 features — WebSocket, messaging, streaming, subagents, approvals, tasks, sessions
- **P1 (Power User):** 8 features — command palette, model switcher, markdown, files, notifications, diffs, branching
- **P2 (Polish):** 15 features — graphs, cost tracking, themes, PWA, a11y, cron dashboard, file browser, etc.
- **Estimated effort:** ~35 developer-days across 8 weeks
- **Architecture:** Vanilla JS (no React/Vue) — smart choice for 2GB Pi

### 3. Project Context (`idea.md`) ✅
- Running on Raspberry Pi 4 (2GB RAM)
- 20 GodMode prompts defined
- All infrastructure already set up

---

## 🔍 Key Analysis

### What's Great
- **Design is done** — the mockup is beautiful and well-thought-out
- **Architecture is solid** — vanilla JS, no build step, Pi-friendly
- **Plan is comprehensive** — every feature spec'd with details
- **Constraints are clear** — 2GB RAM, no heavy frameworks

### Challenges
1. **35 dev-days is ambitious** — this is a full product, not a side project
2. **OpenClaw Gateway API compatibility** — many endpoints in the plan may not exist yet in OpenClaw
3. **WebSocket protocol** — need to verify OpenClaw's actual WS message format
4. **Single developer** — no team to parallelize the build
5. **Testing on Pi** — need to validate performance on actual hardware

### Critical Questions to Answer First
1. ❓ Does OpenClaw gateway expose a WebSocket endpoint at `/ws`?
2. ❓ What message types does the WS send? (chat, tool_use, stream_chunk, etc.)
3. ❓ Is there a REST API for sessions, models, files, cron?
4. ❓ How does auth work for the web UI? (token? password?)
5. ❓ Where will this UI be hosted? (Pi itself? Tailscale? Separate server?)

---

## 📋 Actionable To-Do List

### Phase 0: Foundation (Day 1-2) 🔧
*Before writing any feature code*

| # | Task | Priority | Est. |
|---|------|----------|------|
| 0.1 | **Research OpenClaw Gateway API** — check docs at `/usr/lib/node_modules/openclaw/docs` for WS protocol, REST endpoints, auth | CRITICAL | 2h |
| 0.2 | **Decompose mockup into file structure** — split the single HTML into modular CSS/JS files per the plan | HIGH | 3h |
| 0.3 | **Set up dev environment** — local file server, live reload, decide hosting (Pi port 3000? Nginx?) | HIGH | 1h |
| 0.4 | **Create `manifest.json`** for PWA basics | LOW | 30m |
| 0.5 | **Add external libs** — download marked.js, prism.js to `/lib/` | MEDIUM | 30m |

### Phase 1: Make It Live (Days 3-7) 🟢
*Core functionality — chat actually works*

| # | Task | Priority | Est. | Depends On |
|---|------|----------|------|------------|
| 1.1 | **WebSocket connection** — connect to gateway, handle auth, reconnect logic, status indicator | CRITICAL | 4h | 0.1 |
| 1.2 | **Message sending** — textarea → WS send → optimistic render | CRITICAL | 3h | 1.1 |
| 1.3 | **Streaming response** — render tokens as they arrive, blinking cursor, auto-scroll | CRITICAL | 4h | 1.1 |
| 1.4 | **Thinking block** — render thinking content from stream, collapsible | HIGH | 2h | 1.3 |
| 1.5 | **Tool call rendering** — show tool name, target, output in collapsible blocks | HIGH | 3h | 1.3 |
| 1.6 | **Subagent cards** — spawn/running/completed states, live timer | HIGH | 4h | 1.3 |
| 1.7 | **Approval system** — show approval bar, approve/deny buttons, send response | HIGH | 3h | 1.1 |
| 1.8 | **Task plan updates** — render task list, animate state changes | MEDIUM | 2h | 1.3 |
| 1.9 | **Session persistence** — save messages to localStorage, load on refresh | MEDIUM | 3h | 1.3 |
| 1.10 | **Error handling** — connection errors, API errors, graceful degradation | HIGH | 2h | 1.1 |

### Phase 2: Power Features (Days 8-14) 🟡
*Makes it actually useful beyond basic chat*

| # | Task | Priority | Est. | Depends On |
|---|------|----------|------|------------|
| 2.1 | **Markdown rendering** — integrate marked.js + prism.js for code blocks | HIGH | 3h | 1.3 |
| 2.2 | **Code block copy button** — click to copy, show "Copied!" feedback | MEDIUM | 1h | 2.1 |
| 2.3 | **Command palette (Ctrl+K)** — fuzzy search overlay for actions | MEDIUM | 4h | 1.1 |
| 2.4 | **Model switcher** — dropdown to change model, show in message | MEDIUM | 3h | 1.1 |
| 2.5 | **Notification toasts** — bottom-right alerts for events | MEDIUM | 2h | 1.1 |
| 2.6 | **Desktop notifications** — browser Notification API for background alerts | LOW | 1h | 2.5 |
| 2.7 | **Session list sidebar** — list sessions, click to switch, new/delete | HIGH | 3h | 1.9 |
| 2.8 | **Keyboard shortcuts** — Enter/Shift+Enter, Ctrl+K, Ctrl+N, etc. | MEDIUM | 2h | — |
| 2.9 | **Theme toggle** — dark/light/OLED switch with CSS variables | LOW | 2h | — |
| 2.10 | **Mobile responsive** — bottom nav, touch targets, swipe gestures | MEDIUM | 4h | — |

### Phase 3: Polish & Advanced (Days 15-28) 🔵
*Differentiation features*

| # | Task | Priority | Est. | Depends On |
|---|------|----------|------|------------|
| 3.1 | **Cost/usage dashboard** — tokens, cost per session, charts | MEDIUM | 4h | 1.1 |
| 3.2 | **System monitor** — CPU, RAM, temp from Pi | LOW | 3h | 1.1 |
| 3.3 | **Quick prompts/templates** — GodMode prompts + custom templates | MEDIUM | 3h | 1.2 |
| 3.4 | **File attachment** — drag-drop, clipboard paste | LOW | 4h | 1.2 |
| 3.5 | **Export sessions** — markdown, JSON download | LOW | 2h | 1.9 |
| 3.6 | **Cron dashboard** — view/create/manage scheduled tasks | LOW | 4h | 1.1 |
| 3.7 | **Workspace file browser** — browse, view, edit files | LOW | 6h | 1.1 |
| 3.8 | **Code diff viewer** — side-by-side, per-hunk actions | LOW | 4h | 2.1 |
| 3.9 | **PWA service worker** — offline caching, installable | LOW | 3h | — |
| 3.10 | **Execution graph** — visual DAG of agent work | LOW | 6h | 1.6 |
| 3.11 | **Multi-channel view** — Telegram/Discord/Web unified | LOW | 4h | 1.1 |
| 3.12 | **Accessibility** — ARIA, keyboard nav, high contrast | MEDIUM | 3h | — |

---

## 🎯 Recommended Approach

### Option A: Full Build (8 weeks, ~140 hours)
Build everything in the plan. Good if you have a developer working on this full-time.

### Option B: MVP First (2 weeks, ~40 hours) ⭐ RECOMMENDED
Build Phase 0 + Phase 1 only. Get a **working** chat UI that connects to OpenClaw. Then iterate.

**MVP deliverable:**
- Open `http://manjushapi:3000` in browser
- Send messages to OpenClaw
- See streaming responses with thinking/tools/subagents
- Approve commands
- Sessions persist across refreshes

### Option C: Hire a Developer
The mockup and plan are so detailed, you could hand this to any frontend developer and they'd know exactly what to build. Budget: ₹40-60K for a freelancer to build the full thing in 4-6 weeks.

---

## 🚀 Immediate Next Steps

1. **Research the Gateway API** — I'll check OpenClaw docs right now to find the actual WebSocket/REST API
2. **Split the mockup** into modular files (Phase 0.2)
3. **Build the WebSocket connection** (Phase 1.1) — this unlocks everything
4. **Test on Pi** — verify performance with real data

---

## 📁 Files in This Repo

| File | What It Is | Status |
|------|-----------|--------|
| `idea.md` | Project context, setup notes, GodMode prompts | ✅ Reference |
| `openclaw-ui-plan.md` | 30-feature enhancement plan | ✅ Blueprint |
| `openclaw-ui-mockup.html` | Static UI mockup | ✅ Design done |
| `ANALYSIS.md` | This file — analysis & to-do | ✅ New |

---

*Pi's analysis: The design work is done beautifully. The plan is over-scoped for a first release — focus on making the chat work first, then layer features. The mockup alone could impress clients if shown as a product demo.* 🥧
