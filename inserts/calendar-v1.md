<!-- OPENPOT INSERT: calendar v1 -->

## OpenPot Calendar Integration

OpenPot has a Calendar tab that displays events in Month and Agenda views. You serve the data. OpenPot renders it.

### Your Responsibility

You must provide a single HTTP endpoint that OpenPot calls:

```
GET /api/calendar/events?start={ISO8601}&end={ISO8601}
```

This endpoint runs on your HTTP server (the same FastAPI/port 8000 server used for Pulse cards and other OpenPot features). If you do not have an HTTP server running, you cannot serve calendar data — mark `calendar` as not installed in `openpot-status.json` and inform the user that calendar requires the HTTP backend.

### Response Schema (CRITICAL — follow exactly)

Return a JSON object with an `events` array:

```json
{
  "events": [
    {
      "id": "string — stable unique ID, NOT a UUID. Use a prefixed string like gcal-abc123 or caldav-evt-456",
      "title": "string — event title, under 60 chars",
      "start_date": "see Date Format Rules below",
      "end_date": "string or null — same format as start_date",
      "is_all_day": true,
      "notes": "string or null — plain text only, strip any HTML before returning",
      "location": "string or null",
      "calendar_name": "string — human-readable calendar name (e.g. 'Work', 'Personal', 'Holidays')",
      "calendar_color": "string — hex color for the calendar's color bar (e.g. '#f83a22')",
      "source": "string — one of the source values below",
      "status": "string — confirmed, tentative, or cancelled"
    }
  ]
}
```

### Date Format Rules (THIS IS THE #1 PITFALL)

| Event Type | start_date / end_date Format | Example |
|---|---|---|
| All-day event | Date-only string, NO time component | `"2026-04-14"` |
| Timed event | ISO 8601 with timezone offset | `"2026-04-14T09:00:00-04:00"` |

OpenPot sends query parameters in ISO 8601 with Z suffix: `?start=2026-04-01T04:00:00Z&end=2026-04-30T04:00:00Z`

Your endpoint MUST accept both `2026-04-01` and `2026-04-01T04:00:00Z` as valid query parameter formats. OpenPot's decoder handles both response formats, but mixing them incorrectly (e.g. putting a time component on an all-day event) will cause decode failures.

If `is_all_day` is `true`, `start_date` MUST be a date-only string. If `is_all_day` is `false`, `start_date` MUST include a time and timezone offset.

### Event Source Values

Use these in the `source` field to indicate where the event came from:

| Source | When to use |
|---|---|
| `google_calendar` | Events from Google Calendar API |
| `apple_calendar` | Events from Apple Calendar / EventKit / icalBuddy |
| `caldav` | Events from a CalDAV server (Nextcloud, Radicale, etc.) |
| `ics` | Events imported from .ics files |
| `agent_cron` | Your own scheduled cron jobs surfaced as calendar events |
| `agent` | Events you created from conversations or observations |
| `agent_detected` | Events you detected in emails, documents, or chat |
| `reminder` | Reminders you created for the user |
| `manual` | Events the user explicitly asked you to create |

### Calendar Provider Setup

When the user asks to set up calendar integration, ask which provider they use. Then follow the appropriate setup path:

**Google Calendar:**
1. Ask the user to create a Google Cloud project and enable the Google Calendar API
2. Ask the user to create OAuth 2.0 credentials (Desktop application type) and provide the `client_id` and `client_secret`
3. Store credentials in your `.env.service` file (or equivalent environment configuration)
4. Install the dependency: `pip install google-api-python-client google-auth-oauthlib`
5. Run the OAuth flow — this will open a browser for the user to authorize access. Store the resulting token.
6. Query ALL calendars via `calendarList().list()` — do NOT query only `'primary'`. Users have work calendars, shared calendars, holiday calendars, and subscriptions. Querying only primary misses most of their events.
7. Use each calendar's `summary` as `calendar_name` and `backgroundColor` as `calendar_color`

**Apple Calendar (macOS agents only):**
1. Install `icalBuddy` via Homebrew: `brew install ical-buddy`
2. The user will be prompted to grant Calendar access (one-time macOS permission)
3. Query all calendars — use each calendar's name and color
4. Alternatively, use Python `EventKit` bindings if available

**CalDAV (Nextcloud, Radicale, Baikal, etc.):**
1. Ask the user for: CalDAV server URL, username, password
2. Install the dependency: `pip install caldav`
3. Store credentials securely in `.env.service`
4. Query all calendars on the account — use each calendar's display name and color

**No calendar provider:**
If the user doesn't use any external calendar, you can still serve calendar data from your own sources: cron job schedules, deadlines detected in conversations, reminders, and manually created events. These all use agent-sourced `source` values.

### Building the Endpoint

Add the endpoint to your FastAPI server (the same one serving Pulse cards). The endpoint must:

1. Accept `start` and `end` query parameters in both date-only and ISO 8601 formats
2. Query the configured calendar provider(s) for events in the requested range
3. Normalize all events to the schema above
4. Strip HTML from the `notes` field (Google Calendar often returns HTML in description fields)
5. Return the `{"events": [...]}` response

After the endpoint is working, update `openpot-status.json` to mark calendar as installed:

```json
{
  "calendar": {
    "version": 1,
    "installed": true,
    "installed_at": "2026-04-10T00:00:00Z",
    "notes": "Google Calendar via OAuth"
  }
}
```

Then restart your HTTP server so the status update is picked up.

### Duplicate Detection

If the user has multiple calendar sources (e.g. Google Calendar AND CalDAV both seeing the same Exchange calendar), the same event may appear from multiple sources. Detect duplicates by matching title + start time + duration within a 5-minute tolerance window. Keep only one copy, preferring the primary source.

### Authorization Model (MANDATORY)

The user decides what goes on the calendar. You read, suggest, and present — but never silently create.

| Source | Default behavior |
|---|---|
| External calendars (Google, Apple, CalDAV) | Always show — user authorized the connection |
| Your cron job schedules | Ask first: "Want your cron schedules on the calendar?" |
| Events you create from conversations | Ask first: "I noticed a deadline. Want me to add it?" |
| Events you detect in emails/documents | Ask first: "Found an event in that email. Add to calendar?" |
| Reminders | Ask first before creating |

Track the user's preferences in your memory:

```
Calendar preferences:
- Show external calendar events: YES
- Show agent cron jobs: [ask on first occurrence]
- Auto-add agent-detected events: NO (ask first)
- Auto-add conversation events: NO (ask first)
```

Never add events to the user's real external calendar (Google, Apple, CalDAV) without explicit confirmation. The Calendar tab is read-only from OpenPot's side — you are rendering, not writing.

### Verification

After setup, test the endpoint:

```bash
curl -s -H "Authorization: Bearer <your-token>" \
  "http://localhost:8000/api/calendar/events?start=2026-04-01T04:00:00Z&end=2026-04-30T04:00:00Z" \
  | python3 -c "import sys,json; d=json.load(sys.stdin); print(f'{len(d[\"events\"])} events')"
```

Then ask the user to force-quit and relaunch OpenPot so it picks up the new `openpot-status.json`. The Calendar tab should show events with colored bars matching each calendar's color.

### Common Pitfalls

1. **Date format mismatch** — all-day events with time components cause decode failures in OpenPot
2. **Only querying primary calendar** — misses work, holiday, and shared calendars
3. **HTML in notes** — Google Calendar returns `<div>` tags in descriptions; strip before returning
4. **Event ID as UUID** — OpenPot expects string IDs like `gcal-abc123`, not bare UUIDs
5. **Env vars lost on restart** — store credentials in `.env.service` or systemd `EnvironmentFile`, never manual `export` commands
6. **Status file not picked up** — restart your HTTP server AND tell the user to force-quit and relaunch OpenPot

<!-- END OPENPOT INSERT: calendar v1 -->
