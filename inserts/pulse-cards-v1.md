<!-- OPENPOT INSERT: pulse-cards v1 — installed by openpot-awareness skill -->
<!-- DO NOT EDIT — managed by the OpenPot Awareness Skill -->

## OpenPot Pulse Cards

You are connected to OpenPot, which has a **Pulse tab** — a proactive
notification surface on the user's phone and iPad. You can deliver
information cards to this surface without the user asking.

Cards appear grouped by category with color-coded strips, unread
indicators, swipe-to-dismiss, and stacking when multiple cards share
a category. Each card shows which agent sent it.

### Card API

**Endpoint:** `POST /api/cards`
**Auth:** Bearer token (same as other API endpoints on port 8000)

**Payload:**

```json
{
  "title": "Card title — keep under 60 characters",
  "body": "Card body — keep under 280 characters",
  "category": "finance",
  "priority": "normal",
  "agent_id": "your-agent-id",
  "expires_at": "2026-04-08T00:00:00Z"
}
```

**Required fields:** title, body, category
**Optional fields:** priority, agent_id, expires_at

### Reading and Dismissing

OpenPot calls these endpoints as the user interacts with cards:

```
GET    /api/cards                    — fetch all active cards
PATCH  /api/cards/{id}/read          — mark a card as read
PATCH  /api/cards/{id}/dismiss       — dismiss a card
```

Your HTTP server must support these routes.

### Categories

Use the correct category. It determines the color strip and grouping.

| Category | Color | Use For |
|----------|-------|---------|
| finance | #30D158 green | Market updates, portfolio, DCA signals, spending |
| calendar | #378ADD blue | Schedule, meetings, deadlines, time-bound events |
| health | #FF6B6B red | Medications, appointments, wellness |
| system | #4ECDC4 teal | Server health, service status, infrastructure |
| email | #7F77DD purple | Important emails, action-needed messages |
| projects | #2AACAC dark teal | Task updates, deadline reminders, milestones |
| entertainment | #FF9F0A orange | Sports scores, content recommendations |
| reminder | #FFD60A yellow | User-requested reminders, follow-ups |
| briefing | #6C7A89 gray | Daily/weekly summaries, scheduled reports |

### When to Create Cards

- After running a cron job — POST a card summarizing the output
- When something time-sensitive happens — deadline, meeting, alert
- When you notice something proactively — insight, recommendation
- When the user asked to be reminded

### When NOT to Create Cards

- Don't card every chat message — cards are for proactive delivery
- Don't duplicate what the user just asked about in chat
- Don't spam — one finance card per day is enough if nothing changed

### Card Expiration

Set `expires_at` for time-sensitive cards. Meeting reminders expire
after the meeting. Daily briefs expire at end of day. Cards without
`expires_at` persist until dismissed.

### Agent ID

Always include `agent_id`. OpenPot shows agent badges on cards when
multiple agents are connected to the same app.

<!-- END OPENPOT INSERT: pulse-cards v1 -->
