<!-- OPENPOT INSERT: pulse-channels v1 -->

## Pulse Channel Awareness

OpenPot organizes your card output into channels based on the `category` field. Each card you POST is automatically filed into a channel matching its category. The user can then configure each channel's delivery mode — whether cards appear in the main stream, only in the channel, or both.

### Canonical Category List

Use consistent category names from this list. Inconsistent categories (e.g. "finance" vs "Finance" vs "investing") create duplicate channels. Always use lowercase.

| Category | Use for |
|---|---|
| `briefing` | Morning briefs, evening summaries, weekly rollups |
| `system` | Health checks, service status, uptime reports |
| `finance` | DCA signals, portfolio updates, market observations |
| `calendar` | Schedule digests, conflict alerts, deadline warnings |
| `projects` | Task updates, milestone tracking, blockers |
| `education` | Learning content, research summaries |
| `health` | Health tracking, medication reminders |
| `entertainment` | Media recommendations, leisure suggestions |

### Creating New Categories

You may create new categories when the user's needs expand into a new domain. OpenPot auto-creates a channel when it sees a category with no matching channel.

When you use a new category for the first time, include a note in that card's body: "I've created a new Pulse channel for [topic]. You can rename or reorganize it in the channel settings."

Do not create categories speculatively. Only create a new category when you have actual content to deliver in that domain.

### The Origin Field

Always include the `origin` field in card POSTs to indicate why the card was generated:

| Origin | When to use |
|---|---|
| `cron` | Output from a scheduled cron job |
| `alert` | Triggered by a threshold or condition being met |
| `agent_initiated` | You are proactively reaching out to the user |
| `announce` | Broadcast or system-wide announcement |

The origin field is informational — OpenPot displays it but does not use it for routing. Routing is based on `category`.

### Delivery Modes

The user can configure each channel to one of two delivery modes:

- **Stream + Channel** — cards appear in the main Pulse stream AND in the channel. This is the default.
- **Channel Only** — cards bypass the main stream and go directly to the channel. The user reviews them on their own schedule.

You do not control delivery modes — the user sets them per channel in OpenPot. However, you should understand the implications:

- Cards in "Channel Only" channels are not lost — they are accessible in the channel view
- The user chose "Channel Only" because they want less noise in their stream for that category
- Do not work around this by changing categories to force cards into the stream

### Priority Override

Cards with `"priority": "high"` always appear in the main stream regardless of the channel's delivery mode. Reserve high priority for items requiring immediate user attention:

- Service outages or critical system failures
- Time-sensitive financial alerts
- Calendar conflicts within the next hour
- Security alerts

Do not use high priority for routine updates, daily briefings, or informational content. Overusing high priority defeats the purpose of the user's delivery mode preferences.

### Card POST Example with Channel Fields

```json
{
  "title": "Morning Brief",
  "body": "3 meetings today, PayDay tomorrow, DCA signals pending review.",
  "category": "briefing",
  "priority": "normal",
  "agent_id": "your-agent-id",
  "origin": "cron",
  "expanded_body": "## Full Morning Brief\n\n### Today's Schedule\n...",
  "actions": ["discuss", "dismiss"]
}
```

### Channel Digest Behavior

If the user has not opened a channel in 48+ hours and unread cards exceed 3, you may compose a single digest card for the stream summarizing what is waiting:

"3 unread in System — all services healthy. 5 unread in Finance — DCA signals pending review."

Use category `briefing` for digest cards. This prevents the "I forgot to check System for two weeks" pattern without cluttering the stream with individual cards.

<!-- END OPENPOT INSERT: pulse-channels v1 -->
