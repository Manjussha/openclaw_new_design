# OpenClaw Agent UI — Complete Enhancement Plan

## Context
Transform the static mockup (`openclaw-ui-mockup.html`) into a fully interactive, real-time web application that connects to the OpenClaw gateway at `192.168.1.5:18789`. The Pi has 2GB RAM — the UI must be lightweight (vanilla JS, no heavy frameworks).

---

## Architecture

### Tech Stack
| Layer | Choice | Why |
|-------|--------|-----|
| Core | Vanilla ES modules | No build step, Pi-friendly |
| WebSocket | Native `WebSocket` API | Zero dependencies |
| Markdown | marked.js (35KB) | Lightweight, fast |
| Syntax Highlighting | Prism.js (11KB core) | Modular, small |
| Charts | uPlot (35KB) | Fastest charting lib |
| Icons | Lucide (SVG sprites) | Replace emoji, crisp at all sizes |
| Storage | localStorage + IndexedDB | Sessions, settings, large data |
| Terminal | xterm.js (optional) | Embedded terminal |
| PWA | Service Worker | Offline + installable |

### File Structure
```
openclaw-ui/
├── index.html                  # App shell
├── manifest.json               # PWA manifest
├── sw.js                       # Service worker
├── css/
│   ├── variables.css           # Theme tokens
│   ├── layout.css              # Sidebar, main, panels, responsive
│   ├── chat.css                # Messages, thinking, tasks, subagents
│   ├── components.css          # Tool blocks, diffs, approvals, modals
│   └── themes/
│       ├── dark.css            # Default (current)
│       ├── light.css           # Light theme
│       ├── oled.css            # Pure black
│       └── nord.css            # Nord palette
├── js/
│   ├── app.js                  # Boot, routing, global state
│   ├── websocket.js            # Gateway WS connection + reconnect
│   ├── chat.js                 # Message rendering + streaming
│   ├── subagents.js            # Subagent cards + tool visualization
│   ├── approvals.js            # Approval workflow handling
│   ├── commands.js             # Command palette + slash commands
│   ├── sidebar.js              # Navigation, sessions, channels
│   ├── dashboard.js            # Right panel stats + live updates
│   ├── settings.js             # Settings modal
│   ├── notifications.js        # Toast + desktop notifications
│   ├── storage.js              # localStorage/IndexedDB persistence
│   ├── markdown.js             # Markdown + code rendering
│   ├── shortcuts.js            # Keyboard shortcut manager
│   ├── files.js                # File browser + editor
│   ├── cron.js                 # Scheduled tasks UI
│   └── export.js               # Export session (MD/PDF/JSON)
├── lib/
│   ├── marked.min.js           # Markdown parser
│   ├── prism.min.js            # Syntax highlighter
│   ├── uplot.min.js            # Charts
│   └── lucide.min.js           # Icons
└── assets/
    ├── icons/                  # SVG icon sprites
    ├── sounds/                 # Notification sounds
    └── logo.svg                # OpenClaw logo
```

---

## P0 — Must Have (Make It Work)

### 1. WebSocket Gateway Connection
**What:** Real-time bidirectional communication with OpenClaw gateway
**Why:** Everything depends on this — chat, streaming, approvals, subagents
- Connect to `ws://192.168.1.5:18789/ws` with auth token header
- Auto-reconnect with exponential backoff (1s → 2s → 4s → 8s → max 30s)
- Connection state machine: `connecting → connected → disconnected → reconnecting`
- Status indicator in topbar (green dot = connected, yellow = reconnecting, red = disconnected)
- Heartbeat ping every 30s to detect stale connections
- Message type routing: `chat`, `thinking`, `tool_use`, `tool_result`, `approval_request`, `task_update`, `subagent_spawn`, `subagent_complete`, `stream_chunk`, `error`
- Message queue for offline — buffer sends, flush on reconnect

### 2. Real Message Sending
**What:** Type and send messages from the web UI to OpenClaw
**Why:** Core interaction — the whole point of the UI
- `Enter` to send, `Shift+Enter` for newline
- Textarea auto-resize as user types (up to max-height)
- Send button disabled while agent is processing (prevent double-send)
- Message appears immediately in chat (optimistic rendering)
- Typing indicator ("OpenClaw is thinking...") while waiting for response
- Cancel button to abort agent mid-response
- Message edit — click to edit your last sent message and re-submit
- Empty message prevention

### 3. Streaming Response Rendering
**What:** Show agent's response token-by-token as it generates
**Why:** Feels alive and fast — users see progress instead of waiting
- Render text chunks as `stream_chunk` events arrive via WebSocket
- Blinking cursor at stream insertion point
- Auto-scroll to bottom during streaming
- **Smart scroll:** pause auto-scroll when user scrolls up, resume on "scroll to bottom" button
- Thinking block appears first (streams its content too)
- Tool use blocks appear dynamically mid-stream
- Subagent cards animate in when spawned
- Final response assembles from all parts
- Handle stream interruptions gracefully (show partial response)

### 4. Dynamic Subagent Rendering
**What:** Subagent cards appear, update, and complete in real-time
**Why:** The core visual differentiator — shows parallel AI work happening
- On `subagent_spawn` event → animate new card into grid
- Card states: `spawning` (fade in) → `running` (cyan border pulse) → `completed` (green) / `failed` (red)
- Live elapsed time counter on running subagents (00:00 → 00:01 → ...)
- Tool calls appear inside cards as they happen
- Each tool call: icon + name + target + duration + collapsible output
- Expand/collapse individual subagent cards
- Collapse all / Expand all toggle
- Failed subagents show error message with stack trace (collapsible)
- Subagent nesting — if a subagent spawns its own subagents, show as indented tree

### 5. Interactive Approval System
**What:** Approve or deny dangerous commands from the UI
**Why:** Security gate — prevents unintended destructive actions
- Approval bar slides in when `approval_request` arrives
- Shows: command text, risk level badge (low/medium/high/critical), requesting subagent name
- Two buttons: Approve (green), Deny (red)
- Countdown timer (default 5 min) — auto-deny on timeout
- "Always approve this command" checkbox → saves to localStorage
- "Always approve from this subagent" option
- Visual feedback: green flash on approve, red shake on deny
- Approval history log (accessible from right panel)
- Sound alert for new approval requests (if enabled)
- Desktop notification for approvals when tab is in background

### 6. Task Plan Live Updates
**What:** Task checklist updates in real-time as agent works
**Why:** Shows progress at a glance — user knows what's happening without reading everything
- Tasks animate from `pending` → `active` (cyan pulse) → `done` (green check)
- Progress bar fills smoothly as tasks complete
- Counter updates: "3 / 7 complete"
- New tasks can be dynamically inserted mid-execution
- Failed tasks show red X with error tooltip
- Skipped tasks show gray with strikethrough
- Click task to scroll to the subagent/tool that executed it
- Collapsible task planner (default expanded)

### 7. Session Persistence & Management
**What:** Save and load conversation history
**Why:** Users need to continue work across browser sessions, review past conversations
- Auto-save messages to IndexedDB on every new message
- Session list in sidebar with: title (auto from first message), date, message count, model used
- Click session to load full conversation with all tool outputs
- New session button (Ctrl+N)
- Delete session (with confirmation modal)
- Search sessions by content
- Session rename (click title to edit)
- Export session (see P2)
- Max 100 sessions stored, auto-prune oldest when limit hit
- Session metadata: total tokens, cost estimate, duration

---

## P1 — High Value (Power User Features)

### 8. Command Palette (Ctrl+K)
**What:** Fuzzy-search overlay for every action in the UI
**Why:** Power users want keyboard-driven workflows, not mouse clicking
- Full-screen overlay with search input
- Fuzzy matching on all available actions
- Categories: Models, Skills, Sessions, Settings, Actions
- Actions available:
  - Switch model: "model claude-opus-4-6", "model gemini-3-flash"
  - Run skill: "skill github", "skill seo-audit"
  - Session: "new session", "clear chat", "delete session"
  - UI: "toggle sidebar", "toggle dashboard", "toggle theme"
  - Export: "export markdown", "export pdf", "export json"
  - System: "restart gateway", "view logs", "system status"
- Recent commands section at top
- Keyboard navigation: arrow keys to select, enter to execute, esc to close
- Results show keyboard shortcut hints where applicable

### 9. Model Switcher
**What:** Pick which AI model to use per-message or globally
**Why:** Different tasks need different models — quick SEO check vs deep code review
- Dropdown in input bar (click brain icon)
- Shows all 9 configured models with:
  - Provider icon (Anthropic, Google, Ollama, etc.)
  - Model name
  - Status dot (connected / disconnected)
  - Cost indicator ($, $$, $$$)
- `@model` syntax in message: "@gemini summarize this" → routes to gemini-3-flash
- Per-message model badge shows which model generated each response
- "Compare" mode: send same prompt to 2 models, show responses side-by-side
- Sticky model choice (stays selected until changed)
- Quick toggle between default and last-used model

### 10. Markdown Rendering Engine
**What:** Rich rendering of agent responses — not just plain text
**Why:** Agent outputs code, tables, lists, headers — they need proper formatting
- Full GitHub-flavored Markdown support via marked.js
- Syntax highlighting in code blocks via Prism.js
  - Languages: JavaScript, HTML, CSS, Python, Bash, JSON, YAML, SQL, TypeScript, Go, Rust
- Copy button on every code block (copies without line numbers)
- Code block header shows language label
- Tables render with styled borders (matching theme)
- Task lists render as checkboxes
- Links open in new tab
- Images render inline (with lightbox on click)
- Blockquotes styled with accent border
- Horizontal rules as subtle dividers
- Inline code with distinct background

### 11. File Attachment & Preview
**What:** Send files to the agent, preview agent-created files
**Why:** Users need to share code, screenshots, configs with the agent
- Drag-and-drop zone on chat area
- Click paperclip button in input bar to browse files
- Paste image from clipboard (Ctrl+V with image)
- Supported previews:
  - Images: inline thumbnail, click to expand
  - Code files: syntax-highlighted preview
  - JSON/YAML: formatted tree view
  - PDF: first page preview
- File sent as base64 or uploaded to gateway
- Download button on agent-generated files (code, sitemap, etc.)
- File size limit indicator (warn > 1MB, block > 10MB)

### 12. Notification System
**What:** Alerts for important events even when UI isn't focused
**Why:** Agent runs long tasks — users need to know when something needs attention
- **Toast notifications** (bottom-right, auto-dismiss after 5s):
  - Approval request (orange, stays until acted on)
  - Task completed (green, auto-dismiss)
  - Error occurred (red, click to see details)
  - Subagent finished (blue, auto-dismiss)
- **Desktop notifications** (browser Notification API):
  - Only when tab is not focused
  - Click notification to focus the tab
  - Request permission on first use
- **Sound alerts** (optional toggle):
  - Soft chime for completion
  - Alert sound for approval requests
  - Error buzzer for failures
- **Notification bell** in topbar:
  - Unread count badge
  - Click to open notification history panel
  - Mark all as read

### 13. Multi-Channel View
**What:** See messages from Telegram, Discord, and Web UI in one place
**Why:** OpenClaw receives messages from multiple channels — need unified view
- Channel tabs in topbar (Telegram | Discord | Web)
- Click tab to filter messages to that channel
- "All Channels" mode — unified inbox with channel badges on each message
- Channel status indicators in sidebar (online/offline/error)
- Per-channel styling:
  - Telegram messages: blue accent
  - Discord messages: purple accent
  - Web UI messages: default accent
- Cross-channel context — agent sees full history regardless of channel
- Unread message counts per channel in sidebar
- Channel-specific settings (notification preferences per channel)

### 14. Enhanced Code Diff Viewer
**What:** Better way to review code changes the agent made
**Why:** Code review is critical — users need to verify changes before approving pushes
- **Two view modes:**
  - Inline diff (current) — additions/deletions in single column
  - Side-by-side diff — old vs new in two columns
- Toggle between views with button
- **Multi-file diff:**
  - File tree sidebar showing all changed files
  - Click file to see its diff
  - Stats: files changed, insertions, deletions
- Per-hunk actions: Accept / Reject / Edit
- Copy full file content (new version)
- Syntax highlighting for 20+ languages
- Line number linking (click to highlight)
- Collapse unchanged sections (show "... 15 lines hidden ...")
- Search within diff

### 15. Conversation Branching & Retry
**What:** Fork or retry conversations from any point
**Why:** Exploration — try different approaches without losing history
- **Retry button** on every agent response:
  - Re-sends the user message, gets a new response
  - Shows both responses with "Version 1 / Version 2" tabs
  - Can retry with a different model
- **Branch button** on any message:
  - Creates a new session forked from that point
  - Branch indicator in sidebar (shows parent session)
  - Branches share history up to fork point
- **Compare view:**
  - Side-by-side comparison of two response versions
  - Highlight differences between responses
- Branch tree visualization in sidebar (optional)

---

## P2 — Nice to Have (Polish & Delight)

### 16. Agent Execution Graph
**What:** Visual flowchart of the agent's execution path
**Why:** Complex tasks spawn many subagents — visualize the orchestration
- Toggle button: "Chat View" ↔ "Graph View"
- DAG (directed acyclic graph) showing:
  - Main agent node at top
  - Subagent nodes branching out
  - Tool call nodes as leaves
  - Edges showing data flow
- Node colors by status: green=done, cyan=running, red=failed, gray=pending
- Click any node → scroll to that section in chat view
- **Timeline/waterfall view:**
  - Horizontal bars showing execution duration
  - Parallel subagents shown as overlapping bars
  - Hover for details (tool name, duration, token count)
- Rendered with SVG (lightweight, no canvas dependency)

### 17. Cost & Usage Dashboard
**What:** Track API spending and usage patterns
**Why:** API costs add up — users need visibility to manage budget
- **Session stats** (already in right panel, make interactive):
  - Tokens: input / output / cached (breakdown)
  - Estimated cost per session
  - Models used with per-model breakdown
- **Daily/weekly/monthly charts:**
  - Bar chart: tokens per day
  - Pie chart: usage by model
  - Line chart: cost trend over time
- **Budget system:**
  - Set monthly budget threshold
  - Progress bar: $12.50 / $50.00 (25%)
  - Alert when approaching 80%, 90%, 100%
  - Optional hard cap: pause requests at limit
- **Export:** Download usage data as CSV
- Data stored in IndexedDB, synced from gateway API

### 18. System Monitor Widget
**What:** Live Raspberry Pi system stats in the dashboard
**Why:** Pi has limited resources — need to know if it's struggling
- Polled from gateway API every 10s
- **Metrics:**
  - CPU usage (4 cores) — bar or sparkline
  - RAM usage — bar with used/free/cached
  - Disk usage — bar with used/free
  - CPU temperature — number with color (green < 60, orange < 80, red > 80)
  - Network: upload/download throughput
  - Gateway uptime
  - Active WebSocket connections count
- **Mini sparklines** in right panel (last 60 data points = 10 min window)
- Click to expand into full system dashboard modal
- Alert toast if CPU > 90% or RAM > 90% or temp > 80C

### 19. Theme Engine
**What:** Multiple visual themes + custom theme builder
**Why:** Everyone has different preferences — dark, light, OLED, etc.
- **Built-in themes:**
  - Midnight (current dark) — default
  - Light — white background, dark text
  - OLED Black — pure #000 background (saves battery on OLED)
  - Nord — blue-gray palette
  - Solarized Dark — warm dark palette
  - Dracula — purple-based dark
- **Custom theme builder:**
  - Color pickers for: background, text, accent, border, success, error, warning
  - Live preview as you adjust
  - Save custom themes with names
  - Import/export theme as JSON
- Auto light/dark based on `prefers-color-scheme`
- Smooth CSS transition on theme switch (300ms fade)
- Theme persisted in localStorage

### 20. Quick Prompts / Template Library
**What:** Pre-built prompt templates for common tasks
**Why:** Speed up repetitive workflows — one click instead of typing
- **Sidebar section** — "Prompts" with collapsible categories:
  - SEO: Audit site, Keyword research, Meta optimization, Sitemap check
  - Web Dev: Code review, Bug fix, Feature build, Deploy
  - Strategy: Content calendar, Competitor analysis, Marketing plan
  - GodMode: All 20 prompts from idea.md pre-loaded
- **Template variables:**
  - `{{url}}` — prompts for URL before sending
  - `{{repo}}` — prompts for repository
  - `{{topic}}` — prompts for topic
  - Fill-in modal appears when template has variables
- **Custom templates:**
  - Save any message as template (right-click → "Save as template")
  - Edit template name, category, variables
  - Reorder templates via drag-and-drop
- **Quick access:** Type `/` in input bar to see template suggestions

### 21. Keyboard Shortcuts
**What:** Full keyboard-driven control of the UI
**Why:** Efficiency — power users never want to touch the mouse
- **Global shortcuts:**
  | Key | Action |
  |-----|--------|
  | `Ctrl+K` | Open command palette |
  | `Ctrl+N` | New session |
  | `Ctrl+B` | Toggle sidebar |
  | `Ctrl+/` | Toggle right panel |
  | `Ctrl+L` | Clear chat |
  | `Ctrl+,` | Open settings |
  | `Ctrl+Shift+T` | Toggle theme (dark ↔ light) |
  | `Esc` | Close modal / cancel action |
  | `↑` (in empty input) | Edit last message |
  | `Ctrl+Shift+C` | Copy last code block |
  | `Ctrl+Enter` | Force send (even mid-stream) |
  | `?` | Show shortcut cheatsheet |
- **Chat navigation:**
  | Key | Action |
  |-----|--------|
  | `Home` | Scroll to top |
  | `End` | Scroll to bottom |
  | `Page Up/Down` | Scroll by page |
  | `J/K` | Next/previous message (vim-style) |
- Shortcut cheatsheet modal (press `?`)
- Customizable shortcuts in settings

### 22. Export & Sharing
**What:** Export conversations in multiple formats
**Why:** Documentation, sharing with team, archiving
- **Export formats:**
  - **Markdown** — clean .md with headers, code blocks, tool outputs
  - **PDF** — print-friendly layout with syntax highlighting
  - **JSON** — full session data (messages, tools, subagents, timing, tokens)
  - **HTML** — standalone self-contained HTML (like current mockup but with real data)
- **Selective export:**
  - Export entire session or selected messages only
  - Include/exclude: thinking blocks, tool outputs, subagent details
- **Share:**
  - Generate shareable link (if gateway supports it)
  - Copy session summary to clipboard (markdown formatted)
- **Auto-export:**
  - Optional: auto-save every session as .md to workspace directory
  - Configurable in settings

### 23. Audit Log Viewer
**What:** Full searchable log of all agent actions
**Why:** Security and debugging — know exactly what the agent did
- **Log panel** (accessible from topbar "Logs" button)
- **Log entries include:**
  - Timestamp
  - Action type: tool_call, approval, command_exec, file_write, git_push, error
  - Actor: main agent or subagent name
  - Details: command text, file path, error message
  - Duration
  - Status: success / failed / denied
- **Filters:**
  - By type (tool calls, approvals, errors, etc.)
  - By time range (last hour, today, this week, custom)
  - By status (success, failed, denied)
  - By subagent
- **Search:** Full-text search across all log entries
- **Export:** Download filtered logs as JSON or CSV
- **Visual timeline:** horizontal bar chart showing activity density over time

### 24. Progressive Web App (PWA)
**What:** Install the UI as a native-feeling app
**Why:** Quick access, offline viewing, push notifications
- **manifest.json:**
  - App name: "OpenClaw"
  - Theme color matching current theme
  - Icons at all required sizes (192, 512)
  - Display: standalone
  - Start URL: index.html
- **Service Worker:**
  - Cache app shell (HTML, CSS, JS, fonts)
  - Offline mode: browse cached sessions, show "reconnecting" for new messages
  - Background sync: queue messages sent offline, send when back online
  - Push notifications: gateway pushes approval requests even when app is closed
- **Install prompt:**
  - Custom "Install App" banner on first visit
  - Install button in settings
- **Mobile optimizations:**
  - Bottom navigation bar (sidebar becomes bottom tabs)
  - Swipe right to open sidebar
  - Swipe left to open dashboard
  - Larger touch targets for approval buttons (48px minimum)
  - Pull-to-refresh

### 25. Accessibility (a11y)
**What:** Make the UI usable by everyone regardless of ability
**Why:** Inclusivity and usability — also improves keyboard navigation for all users
- **ARIA attributes:**
  - `role="log"` on chat area
  - `aria-live="polite"` on streaming text
  - `aria-expanded` on collapsible sections
  - `role="alert"` on approval requests
  - Proper labeling on all buttons and inputs
- **Keyboard navigation:**
  - Tab through all interactive elements in logical order
  - Enter/Space to activate buttons
  - Arrow keys in menus and lists
  - Focus trap in modals
- **Screen reader:**
  - Announce new messages
  - Announce subagent status changes
  - Announce approval requests
  - Meaningful alt text on all visual elements
- **Visual accessibility:**
  - High contrast theme option (WCAG AAA)
  - Minimum 4.5:1 contrast ratio on all text
  - Focus indicators (2px solid outline) on all interactive elements
  - No information conveyed by color alone (always use text/icon too)
- **Motion:**
  - `prefers-reduced-motion` media query respected
  - Toggle to disable all animations in settings
  - No auto-playing animations that can't be paused

### 26. Skill Browser & Manager
**What:** Browse, install, configure skills from the UI
**Why:** Skills are how OpenClaw learns new abilities — needs a good management UX
- **Skill library panel** (modal or sidebar section)
- **Installed skills:**
  - Card per skill: icon, name, description, version, status (enabled/disabled)
  - Toggle switch to enable/disable
  - Settings button → skill-specific configuration panel
  - Uninstall button with confirmation
  - Usage stats: how many times used, last used date
- **Browse ClawHub:**
  - Search available skills
  - Category filter: coding, seo, social, automation, analytics
  - Skill detail page: description, screenshots, install count, rating
  - One-click install (calls `openclaw skills install <name>`)
- **Skill configuration:**
  - Dynamic form generated from skill's config schema
  - Validate inputs before saving
  - Reset to defaults button

### 27. Workspace File Browser
**What:** Browse and edit files in the OpenClaw workspace directory
**Why:** Agent works with files — users need to see and edit them too
- **File tree panel** (toggled from sidebar or right panel)
- **Features:**
  - Directory tree with expand/collapse
  - File icons by type (JS, HTML, CSS, JSON, image, etc.)
  - Git status overlay: M (modified), A (added), ? (untracked), D (deleted)
  - Click file to preview in read-only viewer
  - Edit button → simple code editor (textarea with monospace font + line numbers)
  - Create new file/folder
  - Rename (F2)
  - Delete with confirmation
  - Right-click context menu
- **Preview types:**
  - Code: syntax highlighted
  - Images: inline preview
  - JSON/YAML: formatted tree
  - Markdown: rendered
- **Integration:**
  - "Open in chat" — send file content to agent
  - "Ask agent about this file" — prefills prompt with file context

### 28. Agent Memory & Context Viewer
**What:** Show what context the agent is working with
**Why:** Transparency — users need to understand what the agent "knows"
- **Context meter:**
  - Visual bar: tokens used / context window max (200,000 for Opus)
  - Color-coded: green (< 50%), yellow (50-80%), red (> 80%)
  - Breakdown: system prompt, conversation, tool outputs, thinking
- **Memory panel:**
  - Show key facts agent has extracted from conversation
  - Pinned items that persist across sessions
  - Boot prompt / system prompt viewer (read-only)
  - Agent personality / persona viewer
- **Context overflow warning:**
  - Alert when approaching context limit
  - Suggest: "Start new session" or "Summarize and continue"
  - Auto-compaction indicator (when gateway compacts context)

### 29. Scheduled Tasks / Cron Dashboard
**What:** View and manage OpenClaw cron jobs
**Why:** 24/7 agent needs scheduled tasks — SEO checks, content posting, monitoring
- **Cron panel** (accessible from sidebar or command palette)
- **Task list:**
  - Name, schedule (human-readable: "Every day at 9 AM"), status, next run
  - Toggle enable/disable
  - Edit schedule / prompt
  - Delete with confirmation
- **Create new cron job:**
  - Natural language schedule input ("every Monday at 10am")
  - Or cron syntax input (`0 10 * * 1`)
  - Prompt template for what the agent should do
  - Model selection
  - Channel for output (Telegram, Discord, none)
- **Execution history:**
  - List of past runs with: time, duration, status, output preview
  - Click to see full execution log (tool calls, responses)
  - Re-run button
- **Calendar view:**
  - Monthly calendar showing scheduled tasks as dots
  - Click date to see tasks for that day

### 30. Multi-User Support
**What:** Multiple users with separate settings and sessions
**Why:** Teams sharing one OpenClaw instance need isolation
- **User profiles:**
  - Avatar, name, role (admin/user)
  - Login via gateway auth (password or token)
- **Per-user features:**
  - Separate session history
  - Personal settings (theme, shortcuts, notifications)
  - Personal prompt templates
- **Admin panel:**
  - Manage users (add, remove, change role)
  - View all users' activity logs
  - Set global settings
  - Usage quotas per user
- **Activity feed:**
  - Who sent what, when
  - Admin can see all channels' activity

---

## Implementation Phases

### Phase 1: Make It Live (Weeks 1-2)
Core WebSocket connection and real-time chat.
| # | Task | Effort |
|---|------|--------|
| 1 | Decompose mockup into modular file structure | 1 day |
| 2 | WebSocket connection + reconnect logic | 1 day |
| 3 | Real message sending from input bar | 0.5 day |
| 4 | Streaming response rendering | 1 day |
| 5 | Dynamic subagent card rendering | 1 day |
| 6 | Interactive approval system | 1 day |
| 7 | Task plan live updates | 0.5 day |
| 8 | Basic session persistence (localStorage) | 1 day |

### Phase 2: Power Features (Weeks 3-4)
Features that make it genuinely useful beyond basic chat.
| # | Task | Effort |
|---|------|--------|
| 9 | Command palette (Ctrl+K) | 1 day |
| 10 | Model switcher UI | 0.5 day |
| 11 | Markdown + syntax highlighting | 1 day |
| 12 | File attachment & preview | 1 day |
| 13 | Notification system (toasts + desktop) | 1 day |
| 14 | Multi-channel view | 1 day |
| 15 | Enhanced code diff viewer | 1 day |
| 16 | Conversation retry/branching | 1 day |

### Phase 3: Polish & Scale (Weeks 5-8)
Refinement, analytics, and advanced features.
| # | Task | Effort |
|---|------|--------|
| 17 | Execution graph visualization | 2 days |
| 18 | Cost & usage dashboard | 1 day |
| 19 | System monitor widget | 1 day |
| 20 | Theme engine + custom themes | 1 day |
| 21 | Quick prompts / template library | 1 day |
| 22 | Keyboard shortcuts system | 0.5 day |
| 23 | Export (MD/PDF/JSON) | 1 day |
| 24 | Audit log viewer | 1 day |
| 25 | PWA + mobile optimizations | 1 day |
| 26 | Accessibility (a11y) | 1 day |
| 27 | Skill browser & manager | 1 day |
| 28 | Workspace file browser | 2 days |
| 29 | Context/memory viewer | 0.5 day |
| 30 | Cron dashboard | 1 day |
| 31 | Multi-user support | 2 days |

---

## API Endpoints Needed (from OpenClaw Gateway)

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/ws` | WebSocket | Real-time chat, streaming, events |
| `/api/sessions` | GET | List all sessions |
| `/api/sessions/:id` | GET/DELETE | Get or delete session |
| `/api/models` | GET | List configured models + status |
| `/api/models/switch` | POST | Change default model |
| `/api/skills` | GET | List installed skills |
| `/api/skills/install` | POST | Install skill from ClawHub |
| `/api/skills/:name/config` | GET/PUT | Skill configuration |
| `/api/cron` | GET/POST | List/create cron jobs |
| `/api/cron/:id` | PUT/DELETE | Update/delete cron job |
| `/api/system/status` | GET | CPU, RAM, disk, temp, uptime |
| `/api/files` | GET | List workspace files |
| `/api/files/:path` | GET/PUT/DELETE | Read/write/delete file |
| `/api/logs` | GET | Audit log entries (filterable) |
| `/api/approvals/:id` | POST | Approve/deny pending request |
| `/api/auth/profiles` | GET | List auth profiles |
| `/api/usage` | GET | Token/cost usage stats |

---

## Verification Checklist
- [ ] WebSocket connects to gateway and shows green status
- [ ] Send a message → see streaming response with thinking block
- [ ] Subagent cards appear dynamically when spawned
- [ ] Approval bar appears and approve/deny works
- [ ] Task plan updates in real-time
- [ ] Sessions persist across browser refresh
- [ ] Ctrl+K opens command palette
- [ ] Model switcher works
- [ ] Code blocks have syntax highlighting + copy button
- [ ] Notifications appear (toast + desktop)
- [ ] Theme switch works (dark ↔ light)
- [ ] Mobile layout works (bottom nav, swipe gestures)
- [ ] PWA installable on mobile
- [ ] All keyboard shortcuts functional
- [ ] Accessibility: full keyboard navigation, screen reader compatible
