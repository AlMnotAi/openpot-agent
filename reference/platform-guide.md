# OpenPot Platform Guide

**Version:** 1.0
**Managed by:** OpenPot Awareness Skill
**Read this file** before creating Pulse cards, web apps, or expanded card content.

---

## 1. Pulse Cards

Pulse cards are proactive notifications you push to the user's OpenPot app. They appear in the Pulse tab as a card stream.

### When to Create a Card

- Scheduled output (cron jobs): morning briefs, DCA signals, health checks
- Threshold alerts: a metric crossed a boundary the user cares about
- Proactive observations: something changed that the user should know
- Digests: summaries of activity over a time period

Do NOT create a card for content the user asked for in the current conversation. That belongs in chat.

### Card API

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

### Body Text Rule

The `body` field is ALWAYS a complete thought. Never a sentence fragment
that leaves the user wondering what the rest says. OpenPot may truncate
long body text visually, but the user can tap to unfold it. If the body
is cut off mid-sentence, the card feels broken.

- Short notifications (1-2 lines): write the complete message
- Medium content (4-10 lines): write the full text — OpenPot unfolds it inline
- Reports with tables/sections: keep body as 1-2 line summary, put detail in expanded_body

### Minimal Card Example

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

### Report Card Example (with expanded detail)

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

### Canonical Category List

Use these exact strings. Inconsistent casing or synonyms create duplicate channels.

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

You may create new categories when the user's needs expand into a new domain. When you do, include a note in your first card: "I've created a new Pulse channel for [topic]. You can rename or reorganize it."

---

## 2. Expanded Cards (Tap-to-Open Detail)

When a card represents a report (multi-item analysis, signal breakdown, diagnostic summary), include an `expanded_body` field with the full markdown content.

### Rules

- Keep `body` as a 1-2 line summary (the compact card preview).
- Put the full analysis in `expanded_body` using markdown: headers, tables, bullet points, bold.
- Include `"actions": ["discuss", "dismiss"]` for report cards.
- Cards without `expanded_body` are glanceable only — they show title + body and cannot be tapped open.

### Cards That Should Have expanded_body

- DCA signal reports → full signal table with tickers, prices, signals, conviction
- Morning briefs → full briefing with calendar, weather, tasks, news
- System health → full diagnostic with all service statuses, uptime, memory, disk
- Any multi-item analysis or report

### Cards That Should NOT Have expanded_body

- Simple reminders ("Oil change due")
- Single-fact notifications ("Market closed today")
- Calendar alerts ("Meeting in 30 minutes")

### Formatting Guidelines for expanded_body

- Use `##` headers to separate report sections.
- Use markdown tables for structured data (tickers, metrics, comparisons).
- Use bold for emphasis on key values.
- Keep total expanded_body under 4,000 characters — this renders in a native scroll view, not a browser.

### Action Buttons

The `actions` array determines which buttons appear at the bottom of the expanded detail view.

| Action | Button Label | Behavior |
|--------|-------------|----------|
| `"discuss"` | "Discuss with [Agent]" | Opens chat with the card content as context |
| `"dismiss"` | "Dismiss" | Closes the detail view and dismisses the card |
| `"acknowledge"` | "Got it" | Marks the card as read and closes |
| `"snooze"` | "Snooze" | Dismisses temporarily, resurfaces later |

---

## 3. Web Apps

Web apps are persistent HTML tools that live in the user's Apps tab. Unlike cards (ephemeral notifications), apps are tools the user returns to repeatedly.

### When to Build an App

Only when the user explicitly requests a persistent tool. Examples:
- "Build me a medication tracker"
- "I need a DCA calculator"
- "Can you make a unit converter?"

Never build an app speculatively. Always confirm with the user before building.

### App Types

| Type | Description | Backend needed? |
|------|-------------|-----------------|
| Type 1: Static tool | Calculator, converter, reference — pure HTML/CSS/JS, no server state | No |
| Type 2: Smart app | Tracker, dashboard — needs persistent data, reads/writes via backend API | Yes (FastAPI service) |
| Type 3: Connected app | Integrates with external APIs (weather, stocks, calendar) via the agent's backend | Yes (proxy through agent server) |

### File Rules

- One self-contained HTML file per app. All CSS and JS inline — no external dependencies except CDN libraries.
- Filename: lowercase, hyphenated, descriptive. Example: `medication-tracker.html`
- Title: set in `<title>` tag. This appears in the Apps tab.
- The file is served by your HTTP server and fetched by OpenPot via `GET /api/apps` (listing) and direct URL (content).

### Design Guidelines

- **Dark theme default.** Background: `#1A1D28` or similar dark. Text: white/light gray.
- **Mobile-first.** The app renders in a WKWebView on iPhone and iPad. Design for touch, not mouse.
- **No scrollbars.** Use CSS `overflow: auto` with `-webkit-overflow-scrolling: touch`.
- **Card aesthetic.** Rounded corners (12-16pt), subtle borders, consistent with OpenPot's dark UI.
- **Gold accent.** Use a warm gold (`#C9A84C` or similar) for primary actions and highlights.
- **Readable typography.** Minimum 16px body text. No tiny fonts.
- **No external images.** Use inline SVG or CSS-drawn elements. The app must work offline once loaded.

### Before You Build — Confirmation Step

Before creating any app, confirm with the user:
1. What the app does (one sentence)
2. Which type (1, 2, or 3)
3. The filename you will use
4. The category (from the canonical list: tools, health, finance, projects, monitoring, entertainment)

### Starter Apps

Three apps ship with the OpenPot Awareness Skill as examples:
- `about-openpot.html` — informational page about the platform
- `weather.html` — weather display (Type 3, connected)
- `market-dashboard.html` — market overview (Type 3, connected)

These demonstrate the design standard. Match their aesthetic when building new apps.

### App API Response Format

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

Each object must include `filename` and `title`. The `description` and `category` fields are optional but strongly recommended.

---

## 4. Decision Framework — Chat vs. Card vs. App

| Situation | Surface | Why |
|-----------|---------|-----|
| User asked a question | Chat | They're in a conversation. Answer there. |
| Cron job produced output | Card | User didn't ask. Push it to Pulse. |
| Alert threshold crossed | Card | Proactive. User needs to know. |
| User asked for a persistent tool | App | They want something that lives in their toolkit. |
| User asked about a previous card | Chat | They're referencing it in conversation. |
| Scheduled digest or rollup | Card | Proactive summary, not a conversation. |

**When in doubt, use chat.** Cards and apps are for specific use cases. Chat is the default.

---

## 5. Calendar Integration

OpenPot has a Calendar tab that displays events from `GET /api/calendar/events`. Calendar-category Pulse cards open the calendar view when tapped.

### Calendar Authorization Rule

**Never add events to the user's calendar without explicit permission.** You may read the calendar, summarize it, and present it — but creating, modifying, or deleting events requires the user to say yes first.

### Calendar Cards

When creating a calendar-category card (schedule digest, conflict alert, deadline warning):
- Use `"category": "calendar"`
- Do NOT include `expanded_body` — calendar cards open the calendar view, not a markdown detail
- Include `"actions": ["view_calendar", "dismiss"]`

---

## 6. Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-04-12 | Initial reference file. Extracted from SOUL.md injection. |
| 1.1 | 2026-04-13 | Added body text completeness rule. Renamed from openpot-platform-guide.md. |
