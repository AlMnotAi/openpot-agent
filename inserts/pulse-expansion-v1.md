<!-- OPENPOT INSERT: pulse-expansion v1 — installed by openpot-awareness skill -->
<!-- DO NOT EDIT — managed by the OpenPot Awareness Skill -->

## OpenPot Pulse Card Expansion

OpenPot now supports **tap-to-expand** on Pulse cards. When a card
carries an `expanded_body` field, the user can tap it to reveal the
full report in a detail sheet with rendered markdown. Cards without
`expanded_body` behave as before — glanceable notifications.

### Updated Card Payload

When you POST a card to `/api/cards` that represents a report or
multi-item analysis, add these optional fields:

```json
{
  "title": "Weekly DCA Signals",
  "body": "5 double-down signals: CRCL, AFRM, TSLA, PLTR, ANET",
  "category": "finance",
  "agent_id": "your-agent-id",
  "expanded_body": "## Weekly DCA Signal Report\n\n**Generated:** Monday 4:30 PM ET\n\n| Ticker | Price | Signal | Conviction |\n|--------|-------|--------|------------|\n| CRCL | $2.14 | Double-down | High |\n| AFRM | $41.30 | Double-down | Medium |\n...",
  "actions": ["discuss", "dismiss"]
}
```

### Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| expanded_body | string | No | Full markdown report. When present, the card becomes tappable. Use headers, tables, bold, bullet points. |
| actions | string[] | No | Action buttons in the expanded view. Options: `discuss`, `dismiss`, `acknowledge`, `snooze` |

### Which Cards Get expanded_body

**Include expanded_body on reports:**
- DCA signal reports — full signal table with tickers, prices, signals, conviction, reasoning
- Morning briefs — complete briefing: calendar, weather, tasks, news
- System health reports — all service statuses, uptime, memory, disk, errors
- Any card summarizing a multi-item analysis

**Do NOT include expanded_body on simple notifications:**
- Single-fact alerts ("Market closed today")
- Simple reminders ("Oil change due")
- Calendar alerts ("Meeting in 30 minutes")

### Formatting the expanded_body

The content is rendered as markdown in a native detail sheet. Use:
- `##` headers to section the report
- `**bold**` for emphasis
- `| table | syntax |` for data tables
- Bullet points for lists

The `body` field stays as the 1-2 line summary shown on the card face.
The `expanded_body` is the full story behind the door.

### Action Buttons

| Action | Button Label | Behavior |
|--------|-------------|----------|
| discuss | "Discuss with [Agent]" | Opens chat about this card (future) |
| dismiss | "Dismiss" | Closes and dismisses the card |
| acknowledge | "Got it" | Marks as read |
| snooze | "Snooze" | Hides temporarily |

For report cards, use `["discuss", "dismiss"]`.
For action-required cards, use `["acknowledge", "dismiss"]`.

<!-- END OPENPOT INSERT: pulse-expansion v1 -->
