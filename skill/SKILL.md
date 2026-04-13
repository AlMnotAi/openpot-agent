---
name: openpot-awareness
description: Teaches this agent how to serve content to the OpenPot iOS client — cards, apps, page captures, calendar, voice, and onboarding
emoji: 🫕
version: 5.1.0
homepage: https://openpot.app
---

# OpenPot Awareness Skill

You are connected to **OpenPot** — a native iOS app that serves as a
command center for AI agents. OpenPot has six tabs: **Chat**, **Pulse**
(notification cards), **Calendar**, **Apps**, **Terminal**, and **Agents**.

## Three Output Surfaces

| Surface | When to use | How |
|---------|-------------|-----|
| Chat | Default. Conversations, answers, follow-ups. | Normal chat response |
| Pulse cards | Proactive output: reports, alerts, briefs. | `POST /api/cards` |
| Web apps | Persistent tools the user returns to. | Build HTML, serve via `/api/apps` |

**Default to chat.** Use cards for output the user did not ask for in
the current conversation. Use apps only when the user requests a
persistent tool.

## Decision Framework — Chat vs. Card vs. App

| Situation | Surface | Why |
|-----------|---------|-----|
| User asked a question | Chat | They're in a conversation. Answer there. |
| Cron job produced output | Card | User didn't ask. Push it to Pulse. |
| Alert threshold crossed | Card | Proactive. User needs to know. |
| User asked for a persistent tool | App | They want something that lives in their toolkit. |
| User asked about a previous card | Chat | They're referencing it in conversation. |
| Scheduled digest or rollup | Card | Proactive summary, not a conversation. |

**When in doubt, use chat.** Cards and apps are for specific use cases.

## Triggers

Activate this skill when:

- User says **"OpenPot sync"** — run the sync process (see Sync section)
- User sends a **page capture** (message contains
  `---PAGE CAPTURE CONTEXT---`) — see Page Captures section
- User asks about **setting up OpenPot** — see Onboarding section
- User asks about **calendar**, **voice**, or **building an app** —
  see the relevant section below
- User asks **"what OpenPot features do I support?"** — check status

---

# Pulse Cards

Pulse cards are proactive notifications you push to the user's OpenPot
app. They appear in the Pulse tab as a card stream.

## When to Create a Card

- Scheduled output (cron jobs): morning briefs, DCA signals, health checks
- Threshold alerts: a metric crossed a boundary the user cares about
- Proactive observations: something changed that the user should know
- Digests: summaries of activity over a time period

Do NOT create a card for content the user asked for in the current
conversation. That belongs in chat.

## Card API

**Endpoint:** `POST /api/cards`

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| title | String | Card headline. Under 60 characters. |
| body | String | 1-2 line summary visible on the compact card. |
| category | String | Determines Pulse channel routing. Use canonical list below. |
| agent_id | String | Your agent ID. |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| priority | String | `"normal"` (default) or `"high"`. Reserve high for genuinely urgent items. |
| origin | String | Why this card was created: `"cron"`, `"alert"`, `"agent_initiated"`, `"announce"`. |
| expanded_body | String | Full markdown report. When present, the card is tappable and opens a detail view. |
| actions | [String] | Action buttons in the detail view. Vocabulary: `"discuss"`, `"dismiss"`, `"acknowledge"`, `"snooze"`. |

## Body Text Rule

The `body` field is ALWAYS a complete thought. Never a sentence fragment
that leaves the user wondering what the rest says. If the body is cut
off mid-sentence, the card feels broken.

- Short notifications (1-2 lines): write the complete message
- Medium content (4-10 lines): write the full text — OpenPot unfolds it inline
- Reports with tables/sections: keep body as 1-2 line summary, put detail in expanded_body

## Minimal Card Example

```json
{
  "title": "System Health Check",
  "body": "All services operational. CPU 23%, memory 41%.",
  "category": "system",
  "agent_id": "your-agent-id",
  "priority": "normal",
  "origin": "cron"
}
```

## Report Card Example (with expanded detail)

```json
{
  "title": "Weekly DCA Signals",
  "body": "5 double-down signals, 4 baseline holds.",
  "category": "finance",
  "agent_id": "your-agent-id",
  "priority": "normal",
  "origin": "cron",
  "expanded_body": "## Weekly DCA Signal Report\n\n**Generated:** Monday 4:30 PM ET\n\n| Ticker | Price | Signal | Conviction |\n|--------|-------|--------|------------|\n| CRCL | $2.14 | Double-down | High |\n| AFRM | $41.30 | Double-down | Medium |\n\n4 baseline holds. Market weakness = accumulation window.",
  "actions": ["discuss", "dismiss"]
}
```

## Canonical Category List

Use these exact strings. Inconsistent casing or synonyms create
duplicate channels.

| Category | Use for |
|----------|---------|
| briefing | Morning briefs, evening summaries, weekly rollups |
| system | Health checks, service status, uptime reports |
| finance | DCA signals, portfolio updates, market observations |
| calendar | Schedule digests, conflict alerts, deadline warnings |
| projects | Task updates, milestone tracking, blockers |
| education | Learning content, research summaries |
| health | Health tracking, medication reminders |
| entertainment | Media recommendations, leisure suggestions |

You may create new categories when the user's needs expand. When you
do, include a note in your first card: "I've created a new Pulse
channel for [topic]. You can rename or reorganize it."

## Expanded Cards (Tap-to-Open Detail)

When a card represents a report, include an `expanded_body` field
with the full markdown content.

- Keep `body` as a 1-2 line summary (the compact card preview).
- Put the full analysis in `expanded_body` using markdown.
- Include `"actions": ["discuss", "dismiss"]` for report cards.
- Cards without `expanded_body` are glanceable only.
- Keep total expanded_body under 4,000 characters.

**Cards that SHOULD have expanded_body:** DCA signal reports, morning
briefs, system health diagnostics, any multi-item analysis or report.

**Cards that should NOT:** Simple reminders, single-fact notifications,
calendar alerts.

## Action Buttons

| Action | Button Label | Behavior |
|--------|-------------|----------|
| `"discuss"` | "Discuss with [Agent]" | Opens chat with the card content as context |
| `"dismiss"` | "Dismiss" | Closes the detail view and dismisses the card |
| `"acknowledge"` | "Got it" | Marks the card as read and closes |
| `"snooze"` | "Snooze" | Dismisses temporarily, resurfaces later |

---

# Web Apps

Web apps are persistent HTML tools that live in the user's Apps tab.
Unlike cards (ephemeral notifications), apps are tools the user returns
to repeatedly.

## When to Build an App

Only when the user explicitly requests a persistent tool. Examples:
"Build me a medication tracker," "I need a DCA calculator."
Never build an app speculatively. Always confirm before building.

## App Types

| Type | Description | Backend needed? |
|------|-------------|-----------------|
| Type 1: Static tool | Calculator, converter, reference — pure HTML/CSS/JS | No |
| Type 2: Smart app | Tracker, dashboard — needs persistent data via backend API | Yes |
| Type 3: Connected app | Integrates with external APIs via the agent's backend | Yes |

## File Rules

- One self-contained HTML file per app. All CSS and JS inline.
- Filename: lowercase, hyphenated, descriptive. Example: `medication-tracker.html`
- Title: set in `<title>` tag. This appears in the Apps tab.
- Served by your HTTP server via `GET /api/apps` (listing) and direct URL (content).

## Design Guidelines

- **Dark theme default.** Background: `#1A1D28` or similar dark. Text: white/light gray.
- **Mobile-first.** Renders in a WKWebView on iPhone and iPad. Design for touch.
- **No scrollbars.** Use CSS `overflow: auto` with `-webkit-overflow-scrolling: touch`.
- **Card aesthetic.** Rounded corners (12-16pt), subtle borders.
- **Gold accent.** Use warm gold (`#C9A84C` or similar) for primary actions.
- **Readable typography.** Minimum 16px body text.
- **No external images.** Use inline SVG or CSS-drawn elements.

## Before You Build — Confirmation Step

Before creating any app, confirm with the user:
1. What the app does (one sentence)
2. Which type (1, 2, or 3)
3. The filename you will use
4. The category (tools, health, finance, projects, monitoring, entertainment)

## App API Response Format

Your `GET /api/apps` endpoint should return:

```json
[
  {
    "filename": "medication-tracker.html",
    "title": "Medication Tracker",
    "description": "Track daily medications and schedules",
    "category": "health"
  }
]
```

---

# Page Captures

The user can capture web pages from OpenPot's in-app browser. Captures
arrive as chat messages with the user's note followed by a structured
context block.

## What You Receive

```
[User's note here]

---PAGE CAPTURE CONTEXT---
URL: https://example.com/product
Title: Product Name
Description: Product description text...
Site: Example

Readable Text:
[Up to 4,000 characters of extracted page content]

Tables:
| Header | Header |
|--------|--------|
| Value  | Value  |

Screenshot: ~/.openclaw/workspace/attachments/{uuid}.jpg
---END PAGE CAPTURE CONTEXT---
```

Fields: URL, Title, Description, Site (metadata), Readable Text (main
content up to 4,000 chars), Tables (if present), Screenshot (file path
to captured viewport image, omitted in Data Only mode).

## Processing a Capture

1. **Use the text data first.** URL, title, readable text, and tables
   cover most questions without needing the screenshot.

2. **For visual analysis,** use your `read` tool on the Screenshot
   path. Only do this when the question requires seeing the page.

3. **Write a visual description** for your own records after analyzing.
   Example: "matte black single-handle kitchen faucet, pull-down
   sprayer, modern industrial style." This description is your
   permanent memory of the page. The image file is temporary.

4. **Respond to the user's note.** Keep responses concise — the user
   is in a chat strip inside the browser. If your response needs
   depth, suggest moving to Chat.

5. **If no note was provided,** confirm briefly: "Noted — [one-line
   description]. I can pull this up anytime."

## Saving to Pulse

When the user signals a page has ongoing value — "save this,"
"remember this," "bookmark this" — push a Pulse card:

- title: Your one-line summary
- body: Your visual description + the user's note, combined naturally
- expanded_body: Full details — URL, price/ratings if product, key data
- category: Most appropriate channel
- actions: ["discuss", "dismiss"]

Include the URL as a tappable link. Not every capture becomes a card —
only when the user signals intent to revisit.

## Re-encounters

When the user returns to a previously captured page, do not re-analyze
from scratch. You have your original visual notes, the user's note, the
extracted data, and any conversation that followed. Pick up where you
left off.

## Link Cards

When recommending a web page to the user, use this format:

```
:::link
url: https://example.com/page
title: Page Title
site: Site Name
note: Why this matters — your annotation.
:::
```

OpenPot renders these as tappable cards that open in the in-app browser.

## Card Recategorization

When the user moves a card to a different Pulse channel, you receive
a notification. This is a learning signal. If the user moves multiple
cards of the same type, ask: "Want me to route [type] to [channel]
automatically?" Never change routing without asking.

---

# Calendar

OpenPot has a Calendar tab with native Month and Agenda views. Events
come from `GET /api/calendar/events` on your backend server (port 8000).

## Authorization Rule

**Never add, modify, or delete calendar events without explicit user
permission.** You may read the calendar, summarize it, and present it.
Creating events requires the user to say yes first. Always ask:
"Would you like me to add this to your calendar?"

## Event Format

```json
{
  "id": "string — stable unique ID, not a UUID",
  "title": "string — under 60 chars",
  "start_date": "ISO 8601 or date-only string",
  "end_date": "ISO 8601 or date-only (optional)",
  "is_all_day": true,
  "notes": "string or null — plain text, no HTML",
  "location": "string or null",
  "calendar_name": "string — human-readable calendar name",
  "calendar_color": "string — hex color",
  "source": "string — google_calendar, agent, etc.",
  "status": "confirmed | tentative | cancelled"
}
```

### Date Format Rules

| Event Type | start_date Format | Example |
|------------|-------------------|---------|
| All-day | Date-only string | `"2026-04-14"` |
| Timed | ISO 8601 with timezone offset | `"2026-04-14T09:00:00-04:00"` |
| Query param | ISO 8601 with Z | `"2026-04-01T04:00:00Z"` |

## Calendar Pulse Cards

When creating a calendar-category Pulse card:
- Use `"category": "calendar"`
- Do NOT include `expanded_body` — calendar cards open the calendar view
- Include `"actions": ["view_calendar", "dismiss"]`

## Common Pitfalls

1. **String IDs, not UUIDs.** UUID objects cause decode failures on iOS.
2. **Timezone offsets required on timed events.** Omitting the offset
   causes events to render at wrong times.
3. **All-day events use date-only strings.** Do not add `T00:00:00`.
4. **Calendar color must be a hex string.** Format: `"#f83a22"`.
5. **Notes field: plain text only.** HTML renders as raw tags.
6. **Do not query only 'primary' calendar.** Query all accessible
   calendars for complete coverage.
7. **The endpoint must handle both date formats as query params.**
8. **Restart the backend server after configuration changes.**
9. **Google OAuth tokens expire.** If events stop appearing, re-run
   the OAuth flow.

## Calendar Setup Steps

1. Ensure your backend server has a `/api/calendar/events` endpoint
2. Configure Google Calendar OAuth credentials
3. Store tokens securely (not in SOUL.md or version control)
4. Test: `curl -H "Authorization: Bearer <token>" http://localhost:8000/api/calendar/events?start=2026-04-01&end=2026-04-30`
5. Update `openpot-status.json` with calendar feature as installed
6. Restart the backend server

---

# Voice

OpenPot supports voice input (speech-to-text) and voice output
(text-to-speech). Voice is entirely client-side — no server changes
are required.

## How It Works

- **Input:** Apple Speech recognition on the device. The user speaks
  and the transcribed text is sent as a normal chat message. You
  receive it as text — no special handling needed.

- **Output:** ElevenLabs text-to-speech on the device. OpenPot reads
  your response aloud. Write naturally, avoid special formatting for
  TTS. If the user is using voice, prefer shorter, conversational
  responses.

## Voice Setup (User-Side, in the OpenPot App)

Walk the user through these steps:

1. **ElevenLabs API Key:**
   Settings → General → Voice Output → ElevenLabs API Key
   (from elevenlabs.io → Profile → API Keys)

2. **Voice Selection (per agent):**
   Agents tab → tap agent → Voice section → Voice ID
   (browse at elevenlabs.io/voice-library, copy the Voice ID)

3. **Voice Model:** Default is `eleven_multilingual_v2`. Can be
   changed per-agent.

4. **Speed:** Adjustable per-agent. Default 1.0, range 0.5–2.0.

## Common Voice Issues

- **"Voice not working"** → check ElevenLabs API key in Settings,
  check API key has credits remaining.
- **"Wrong voice"** → check per-agent Voice ID in Agents tab.
- **"Talk Mode breaks messages"** → do NOT add `operator.talk.secrets`
  scope to gateway config. It interferes with message delivery.

## No Server Changes Needed

Voice requires zero backend configuration. Do not modify SOUL.md,
install services, or configure endpoints for voice. Mark voice as
installed in `openpot-status.json` after the user confirms credentials.

---

# Onboarding

When the user asks to set up or connect OpenPot ("I have OpenPot, get
me set up," "help me connect OpenPot," "OpenPot setup"), follow these
steps.

## Step 1 — Provide Connection Details

You know your own infrastructure. Provide:

1. **Gateway URL** — format: `ws://<your-ip>:18789`
   If Tailscale is available, provide both addresses:
   - LAN: `ws://<lan-ip>:18789` (faster, home WiFi only)
   - Tailscale: `ws://<tailscale-ip>:18789` (works from anywhere)

2. **Bearer token** — send in a private/1:1 channel only.
   Never post tokens in group channels.

3. **Instructions:** "Open OpenPot → Agents tab → Add Agent → paste
   the gateway URL and token → tap Connect."

## Step 2 — Check Tailscale

Run `tailscale status` to check if Tailscale is installed.
- If running: provide the Tailscale IP for the second route.
- If not installed: LAN-only works on home WiFi. For remote access,
  point them to tailscale.com/download.

## Step 3 — Auto-Approve Device Pairing

Tell the user: "When you tap Connect, I'll approve the pairing
automatically. Go ahead."

Then:
1. Run `openclaw devices list` to check for pending requests
2. Approve the request from "openclaw-ios" or "OpenPot":
   `openclaw devices approve <requestId>`
3. Confirm: "You're approved and connected!"

**Security rules:**
- Only auto-approve during an active onboarding conversation
- Only approve ONE device per onboarding flow
- If multiple requests arrive, approve only the first and alert the user
- Outside onboarding, surface pairing requests for manual approval

## Step 4 — Post-Connection

Once connected:

1. Send a welcome Pulse card (if you have an HTTP server):
   title: "Welcome to OpenPot", body: "Your connection is live.
   Explore all six tabs.", category: "system",
   actions: ["acknowledge"]

2. Briefly describe the tabs: Chat, Pulse, Calendar, Apps, Terminal,
   Agents.

3. Offer to run "OpenPot sync" to configure all features.

## Step 5 — Backend Setup (if applicable)

If you have a backend server on port 8000, check health and provide
the server URL and token for OpenPot Settings → Server Configuration.
If you don't have a backend server, skip — Chat, Terminal, and basic
features work with the gateway alone.

## Troubleshooting

- "Connecting stays amber" → check gateway is running, port 18789,
  address starts with ws://, user is on correct network
- "Device not approved" → run `openclaw devices list`, approve manually
- "Pulse is empty" → run "OpenPot sync", check for HTTP server
- "Can't connect on cellular" → Tailscale needed for remote access
- "Agent name blank" → check IDENTITY.md in workspace

---

# Sync Process

When the user says "OpenPot sync":

## Step 1 — Pull the full OpenPot support repo

**Use your shell/bash tool** (not web_fetch or web_search) to clone
the full OpenPot repository. This gives you starter apps, status
templates, and documentation:

```bash
git clone https://github.com/AlMnotAi/openpot-agent.git ~/.openclaw/workspace/openpot-agent
```

If you already have it, pull updates instead:

```bash
cd ~/.openclaw/workspace/openpot-agent && git pull origin main
```

This repo contains:
- `apps/` — starter web apps to install
- `templates/` — status file templates
- `docs/` — setup documentation

## Step 2 — Check current status

Read `openpot-status.json` from your workspace root. If it doesn't
exist, this is a first-time setup — treat all features as version 0.
Use the template from `openpot-agent/templates/` if available.

## Step 3 — Check infrastructure and report

For each feature, check if your infrastructure meets the prerequisites
and report what's available and what's missing:

- **Chat** — always available (gateway only)
- **Pulse Cards** — requires HTTP server with `POST /api/cards`
- **Calendar** — requires HTTP server + Google OAuth + `/api/calendar/events`
- **Voice** — configured in the app, no server needed
- **Page Capture** — always available (gateway only)
- **Apps** — requires HTTP server + `/api/apps` endpoint
- **Terminal** — always available (gateway only)

Offer to help set up missing prerequisites.

## Step 4 — Install starter apps

If the repo was cloned successfully, copy apps from
`openpot-agent/apps/` to your apps directory. Track installed and
user-removed apps in `openpot-starter-apps.json` so removed apps
don't come back on future syncs.

## Step 5 — Write status

Update `openpot-status.json` in your workspace root with current
feature versions and timestamps.

## Step 6 — Report

```
OpenPot Sync Complete
━━━━━━━━━━━━━━━━━━━━
✅ Chat (v1)
✅ Pulse Cards (v1) — requires HTTP server
✅ Calendar (v2) — requires HTTP server + Google OAuth
✅ Voice (v1) — configured in the app, no server needed
✅ Page Capture (v2) — ready
✅ Onboarding (v2) — ready
⬚ Apps — requires HTTP server + /api/apps endpoint

Starter Apps: {count} installed
Last synced: {timestamp}
```

---

# Migration: Clean Up Old SOUL.md Inserts

If your SOUL.md contains `<!-- OPENPOT INSERT` markers from a previous
version of this skill, remove them. The native skill system loads this
SKILL.md automatically — SOUL.md injection is no longer needed.

To clean up:
1. Read your SOUL.md
2. Delete everything between `<!-- OPENPOT INSERT` and
   `<!-- END OPENPOT INSERT -->` markers (including the markers)
3. Restart the gateway

This is a one-time migration. New installations do not touch SOUL.md.

---

# Rules

- **NEVER modify SOUL.md** — this skill loads natively via OpenClaw
- **NEVER auto-sync** — the user triggers sync with "OpenPot sync"
- **NEVER re-install apps the user deleted** — respect the tracking file
- **NEVER add calendar events without explicit user permission**
- **Always update openpot-status.json** after any change
- App endpoints (`/api/apps`) must not require authentication
