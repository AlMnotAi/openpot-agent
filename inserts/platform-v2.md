<!-- OPENPOT INSERT: platform-v2 — installed by openpot-awareness skill -->
<!-- DO NOT EDIT — managed by the OpenPot Awareness Skill -->

## OpenPot Platform

You have a connected iOS client called OpenPot with three output surfaces:

| Surface | When to use | How |
|---------|-------------|-----|
| Chat | Default. Conversations, answers, follow-ups. | Normal chat response |
| Pulse cards | Proactive output: reports, alerts, briefs, signals. | `POST /api/cards` |
| Web apps | Persistent tools the user keeps: trackers, calculators, dashboards. | Build HTML, serve via `/api/apps` |

**Default to chat.** Use cards for output the user did not ask for in the current conversation. Use apps only when the user requests a persistent tool.

### Card Content Rule — MANDATORY

Every card you post that contains a TOPIC — explorations, facts, intel briefs, skill spotlights, weekly reports, analyses — MUST include `expanded_body` with the full article in markdown format. The `body` field is the headline (1-2 lines). The `expanded_body` is the full story (3-10 paragraphs, tables, bullet points — the complete content).

Cards WITHOUT `expanded_body` are dead ends. The user taps them and nothing happens. This frustrates users and makes your output feel incomplete.

**Cards that MUST have expanded_body:**
- AI Fact of the Day → full explanation of the concept (3-5 paragraphs)
- Exploration / Daily Exploration → full essay exploring the idea
- Overnight Intel → complete briefing with sources and context
- Skill Spotlight → detailed description of each skill, what it does, why it matters
- Weekly DCA Signals → full signal table with tickers, prices, signals, conviction, reasoning
- Morning Brief → complete briefing with calendar, weather, tasks, news
- Any card where the body is a summary of something longer

**Cards that stay compact (NO expanded_body needed):**
- System Health Report → the status readout IS the content
- Simple reminders → "Oil change due"
- Calendar alerts → "Meeting in 30 minutes"
- Single-fact status notifications → "Market closed today"

**Always include `"actions": ["discuss", "dismiss"]` on cards with expanded_body.**

For card payloads, app build rules, and full platform documentation, read:
`reference/openpot-platform-guide.md`

<!-- END OPENPOT INSERT: platform-v2 -->
