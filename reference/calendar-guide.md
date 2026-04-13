# Calendar Guide

This file is read on demand when setting up or troubleshooting the
OpenPot Calendar tab. Do not preload this content.

## Overview

OpenPot has a Calendar tab with native Month and Agenda views. Events
come from `GET /api/calendar/events` on your backend server (port 8000).

## Authorization Rule

**Never add, modify, or delete calendar events without explicit user
permission.** You may read the calendar, summarize it, and present it.
Creating events requires the user to say yes first. Always ask:
"Would you like me to add this to your calendar?"

## Event Format

Events returned by the endpoint must follow this schema:

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
  "calendar_color": "string — hex color for the leading bar",
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

The endpoint must accept both date-only and ISO 8601 query parameters.

## Calendar Pulse Cards

When creating a calendar-category Pulse card (schedule digest, conflict
alert, deadline warning):

- Use `"category": "calendar"`
- Do NOT include `expanded_body` — calendar cards open the calendar
  view, not a markdown detail sheet
- Include `"actions": ["view_calendar", "dismiss"]`

## Common Pitfalls

1. **String IDs, not UUIDs.** Event IDs must be strings. UUID objects
   cause decode failures on the iOS side.

2. **Timezone offsets required on timed events.** Omitting the offset
   causes events to render at wrong times. Always include `-04:00`
   or the appropriate local offset.

3. **All-day events use date-only strings.** Do not add `T00:00:00`
   to all-day events — it causes them to render as timed events.

4. **Calendar color must be a hex string.** Format: `"#f83a22"`.
   Missing or malformed color falls back to gray.

5. **Notes field: plain text only.** HTML in notes renders as raw
   tags in the app. Strip all HTML before returning.

6. **Do not query only 'primary' calendar.** Google accounts often
   have shared calendars, holiday calendars, and subscribed calendars.
   Query all accessible calendars for complete coverage.

7. **The endpoint must handle both date formats as query params.**
   OpenPot sends `start=2026-04-01T04:00:00Z&end=2026-04-30T04:00:00Z`
   for timed ranges. Some backends choke on the `T` in query params.

8. **Restart the backend server after configuration changes.** The
   status endpoint response is cached. Run
   `sudo systemctl restart openpot-server.service` (or your service
   name) after updating `openpot-status.json`.

9. **Google OAuth tokens expire.** If events suddenly stop appearing,
   the refresh token may have expired. Re-run the OAuth flow.

## Setup Steps

1. Ensure your backend server has a `/api/calendar/events` endpoint
2. Configure Google Calendar OAuth credentials
3. Store tokens securely (not in SOUL.md or version control)
4. Test: `curl -H "Authorization: Bearer <token>" http://localhost:8000/api/calendar/events?start=2026-04-01&end=2026-04-30`
5. Update `openpot-status.json` with calendar feature as installed
6. Restart the backend server
