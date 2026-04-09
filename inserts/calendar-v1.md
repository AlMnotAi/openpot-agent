<!-- OPENPOT INSERT: calendar v1 — installed by openpot-awareness skill -->
<!-- DO NOT EDIT — managed by the OpenPot Awareness Skill -->

## OpenPot Calendar Tab

You are connected to OpenPot, which has a **Calendar tab** — a native
Month and Agenda view that displays events you post to the calendar API.
The calendar is read-only for the user. You are the source of truth.

Events you create appear in the user's Calendar tab immediately. Use this
for anything time-bound: meetings, deadlines, reminders, cron job schedules,
maintenance windows, and recurring routines.

### Calendar API

**Endpoint:** `GET /api/calendar/events`
**Auth:** Bearer token (same as other API endpoints on port 8000)

OpenPot fetches this endpoint to populate the Calendar tab. Your HTTP
server must serve a list of events in the following shape.

**Response shape:**

```json
{
  "events": [
    {
      "id": "uuid-or-stable-string",
      "title": "Weekly DCA Review",
      "start_date": "2026-04-14T09:00:00Z",
      "end_date": "2026-04-14T09:30:00Z",
      "is_all_day": false,
      "notes": "Check DCA signals for CRCL, AFRM, TSLA",
      "calendar_color": "#30D158",
      "source": "agent"
    }
  ]
}
```

### Event Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | string | Yes | Stable unique ID. Use a UUID or deterministic slug. Don't change it — OpenPot uses it to track read/dismiss state. |
| title | string | Yes | Short event title. Keep under 60 characters. |
| start_date | string (ISO 8601) | Yes | Event start in UTC. |
| end_date | string (ISO 8601) | No | Event end. Omit for point-in-time events. |
| is_all_day | boolean | No | Defaults to false. Set true for dates without a specific time. |
| notes | string | No | Optional detail shown below the title in the Agenda view. |
| calendar_color | string | No | Hex color for the leading color bar. Defaults to agent gold `#D4A843` if omitted. |
| source | string | No | One of: `agent`, `cron`, `reminder`, `system`, `manual`. Determines the trailing icon in Agenda view. Defaults to `agent`. |

### Source Values and Icons

| Source | Icon | Use For |
|--------|------|---------|
| agent | person.crop.circle | Scheduled by the agent based on context |
| cron | clock.arrow.2.circlepath | Recurring cron jobs |
| reminder | bell | User-requested reminders |
| system | server.rack | Maintenance windows, server events |
| manual | pencil | User-created events the agent is tracking |

### Calendar Colors

Use colors that match the event type. These align with Pulse card categories:

| Color | Hex | Use For |
|-------|-----|---------|
| Agent gold | #D4A843 | General agent-created events (default) |
| Finance green | #30D158 | Financial reviews, DCA schedules |
| Calendar blue | #378ADD | Meetings, appointments |
| Health red | #FF6B6B | Medical appointments, medication schedules |
| System teal | #4ECDC4 | Maintenance windows, server events |
| Projects dark teal | #2AACAC | Project deadlines, milestones |
| Reminder yellow | #FFD60A | User-requested reminders |

### Calendar Views

The user sees two views — both populated from the same `/api/calendar/events` endpoint:

**Month view** — Grid calendar. Days with events show a dot indicator. The user can tap a day to see its events.

**Agenda view** — Chronological list of upcoming events, from today through the end of next month. Events are grouped by day with sticky date headers. All-day events appear first within each day. Events show: color bar, time (or "all-day"), title, notes, and a trailing source icon.

### What to Post

**Post calendar events for:**
- Scheduled cron jobs the user cares about ("Portfolio review runs Monday 4:30 PM")
- Recurring routines you're tracking ("Oil change due April 20")
- Deadlines the agent is managing ("Tax filing deadline April 15")
- Meetings the user mentioned ("Mark call — Friday 2 PM")
- Reminders the user asked the agent to track

**Don't post:**
- Every internal agent operation — only events the user would want to see
- Duplicates of Pulse card reminders for the same event (cards are for the moment, calendar is for planning)
- Past events — keep `/api/calendar/events` pruned of stale entries

### Keeping the Calendar Current

Your HTTP server returns whatever is currently in the events list when OpenPot calls `GET /api/calendar/events`. Keep this list maintained:

- Add events when you schedule them
- Remove past events on a regular cleanup pass
- Update events if the time or details change (same `id`, updated fields)

OpenPot refreshes the calendar when the user opens the tab and on a background poll. Changes appear within one poll cycle.

<!-- END OPENPOT INSERT: calendar v1 -->
