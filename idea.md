# OpenClaw Setup — Project Plan

## Server Details
- **Name:** Manjushapi
- **IP:** 192.168.1.5
- **User:** manjusha
- **Pass:** Manju@805060

## Hardware (Raspberry Pi)
| Spec       | Value                              |
|------------|------------------------------------|
| Model      | Raspberry Pi 4 Model B Rev 1.5     |
| RAM        | 2GB (1.8Gi usable)                 |
| CPU        | 4-core ARM Cortex-A72 (BCM2711)    |
| OS         | Debian 13 (Trixie) — 64-bit        |
| Storage    | 64GB SD card, 52GB free            |
| Desktop    | None (Lite / headless)             |

## What is OpenClaw?
- AI agent that runs 24/7 on your Raspberry Pi
- You talk to it via Telegram / Discord
- It uses cloud AI (Claude + Gemini) to think
- It can execute commands, write code, manage files on the Pi
- It acts as your personal AI assistant for web dev, SEO, and strategy

## Goal
Use OpenClaw as a **Website Developer + SEO Manager + Strategy Manager** for multiple websites.

### What It Will Do
1. **Web Development**
   - Write/edit HTML, CSS, JS code
   - Push code to GitHub
   - Manage deployments across sites
   - Fix bugs, build features on command

2. **SEO Management**
   - Keyword research and suggestions
   - Meta tag optimization
   - Content audits and recommendations
   - Sitemap generation
   - On-page SEO analysis

3. **Strategy Management**
   - Content calendars and scheduling plans
   - Competitor analysis
   - Marketing plans and campaigns
   - Performance reporting and insights

## Configuration
| Component      | Choice                          |
|----------------|---------------------------------|
| AI Providers   | Claude (Anthropic) + Google Gemini |
| Chat Channels  | Telegram + Discord               |
| API Keys       | Ready (both)                     |
| Bot Tokens     | Ready (Telegram + Discord)       |
| Runs 24/7      | Yes, via systemd service         |

## Constraints (2GB RAM)
- No local AI models (Ollama won't work well)
- No heavy browser automation (Puppeteer/Playwright too heavy)
- OpenClaw core + Node.js fits fine (~300-500MB)
- All AI thinking happens in the cloud via API calls

## Installation Steps
1. Install Node.js 22 (via NodeSource — Pi OS ships older versions)
2. Install OpenClaw (`curl -fsSL https://openclaw.ai/install.sh | bash`)
3. Configure AI providers (add Claude + Gemini API keys)
4. Connect Telegram bot (add bot token)
5. Connect Discord bot (add bot token)
6. Set up systemd service for 24/7 auto-start
7. Configure skills for web dev, SEO, and strategy work
8. Security hardening (require approval for commands, firewall)

## Cost
- **Pi electricity:** ~$0.50-1/month (very cheap 24/7)
- **Claude API:** Pay per usage (depends on how much you use)
- **Gemini API:** Has a free tier, then pay per usage
- **Total:** Mostly just API costs, hardware is almost free to run

## Current Setup (Completed)
| Component | Status |
|-----------|--------|
| Node.js 22.22.0 | Installed |
| OpenClaw 2026.2.15 | Installed + running |
| Gateway | `192.168.1.5:18789` (loopback + Tailscale) |
| Tailscale | `https://manjushapi.tailfe47cb.ts.net/` |
| Telegram Bot | @Pimanjushabot — running, polling |
| AI Model | ollama/glm-5:cloud (default) + gemini-3-flash (fallback) |
| Systemd + Lingering | Enabled (24/7 without login) |
| Firewall | SSH (22) + HTTPS (443) + Gateway (18789) |
| Skills | github, coding, discord, gemini, blogwatcher, summarize |
| Auto-approve | Running (Tailscale IPs only) |
| SSH shortcut | `ssh manjushapi` |
| Gateway token | `4324457117b851d5a46d9b2b49b889fd20264b2e2590dcee` |

---

## GodMode Prompt Toolkit (20 Prompts)
*Source: https://lifeongodmode.com/brief*

### Morning Routine

**01 — The CEO Morning Brief**
Prepare a morning brief including: top 3 priorities, calendar scan with context, overnight updates, open loops needing follow-up, and one strategic insight. Keep it concise without unnecessary detail.

**02 — The Energy Architect**
Analyze the daily calendar against peak performance hours (specified by user). For each time block, identify required energy type, whether timing is optimal, and hidden recovery opportunities. Conclude with one recommendation to protect high-value hours.

**03 — The Decision Pre-loader**
For pending decisions listed, provide: the core tension being decided, what you'd advise a close friend, the 10/10/10 test results (satisfaction at 10 minutes, 10 months, 10 years), and a recommendation with reasoning in quick-reference format.

### Email & Communication

**04 — The Inbox Triage Matrix**
Classify emails into four buckets: urgent-important (draft reply), important-not urgent (flag with deadline), delegate (identify person and forwarding note), and archive (explain why). Match response tone to the user's specified communication style.

**05 — The Voice Clone Responder**
Using provided writing examples, draft responses matching the user's exact communication style. Key elements: opening approach, paragraph structure, sign-off format, and tone. Base accuracy on sample emails rather than written descriptions.

**06 — The Follow-Up Tracker**
Review sent messages for open loops: waiting responses (with follow-up drafts), unfulfilled commitments, introduction outcomes, and proposal status. Rank by priority and create an actionable "power hour" list for concentrated follow-up.

### Research & Strategy

**07 — The Meeting Dossier**
Compile pre-meeting research: background and career trajectory, company overview with recent news, digital footprint, mutual connections, likely priorities, three conversation starters, and what both parties might want. Format as a five-minute brief.

**08 — The Competitive Intelligence Scan**
Analyze competitor changes in: product/service updates (90-day window), pricing shifts, team movements, marketing strategy, customer sentiment, exploitable weaknesses, and relative strengths. End with two perspective questions about competitor and competitive strategy.

**09 — The Market Opportunity Radar**
Identify emerging opportunities: adjacent growing markets, disruptive technology, regulatory changes, demographic shifts, competitor exit gaps, and unmet customer demands. Score each by potential impact, effort required, and business fit. Highlight top three with action approaches.

**10 — The Deal Analyzer**
Evaluate opportunities across: upside scenario, realistic downside case, hidden risks and questions to ask, negotiation leverage points, pre-committed walk-away threshold, regret test comparison, and confidence level. Provide direct assessment over diplomatic phrasing.

### Operations & Automation

**11 — The Delegation Architect**
Classify all current tasks into: only-you work, review-required delegations, and fully delegatable items. For delegatable tasks, create assignment briefs, check-in cadence, and completion criteria. Calculate weekly hours freed by delegation.

**12 — The Process Extractor**
Convert a conversational process description into: numbered step-by-step SOP, decision tree logic for judgment calls, quality verification checklist, common failure modes with prevention tactics, and time estimates per step. Format for team member or AI handoff.

**13 — The Weekly Operations Review**
Structure week analysis as: metric scoreboard versus targets, wins and repeatable factors, root causes of misses, blocking issues and solutions, three key priorities for next week, and one recurring pattern worthy of attention. Prioritize clarity over positivity.

### Personal Growth

**14 — The Pattern Interrupt**
Analyze recent decisions and behaviors to identify: underlying patterns you're missing, what you're avoiding, where you're underperforming potential, limiting beliefs driving actions, and one challenging self-directed question. Provide outside perspective without diplomatic softening.

**15 — The Sunday Reset Protocol**
Combine reflection (three wins, one improvement, gratitude, energy/focus/impact ratings) with next-week planning (three non-negotiable rocks, one transformative priority, necessary "no" commitments, forward-looking item). Save as single-page weekly document.

**16 — The Decision Journal**
Document decisions including: surrounding context and triggers, considered alternatives, reasoning for chosen path, expected outcomes and predictions, reversal conditions pre-committed, confidence percentage, and review date. Enable future grading of decision-making quality.

### "God Mode" Prompts

**17 — The Second Brain Sync**
Leverage conversation history to surface: similar past situations and outcomes, previously stated principles applicable now, relationship context for people involved, contradictions with prior commitments, and non-obvious connections across your full context. Synthesize into informed advisor perspective.

**18 — The Board of Advisors**
Model five perspectives on decisions: the systems-focused operator, relationship-focused closer, assumption-challenging contrarian, first-principles philosopher, and your five-year-future self. Allocate two to three sentences per advisor, then synthesize integrated recommendation.

**19 — The Leverage Audit**
Categorize current situation across leverage types: labor (delegated work), capital (invested money), technology (automation), media (content distribution), and network (relationship returns). Identify under- and over-leveraged areas, highest-impact 90-day shift, and overall leverage score.

**20 — The Operating System Prompt**
Reframe AI interaction from conversational tool to autonomous operating system. Specify desired operating characteristics: proactive anticipation, persistent memory, voice matching, challenge-when-needed approach, pattern-finding across domains, background operation, information prioritization, and continuous learning emphasis.

---

## Links
- https://openclaw.ai/
- https://toclawdbot.com/raspberry-pi
- https://n.thenextnewthing.ai/using-a-claude-subscription?source=copy_link
- https://lifeongodmode.com/brief
