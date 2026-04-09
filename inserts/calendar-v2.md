<!-- OPENPOT INSERT: calendar v2 — installed by openpot-awareness skill -->
<!-- DO NOT EDIT — managed by the OpenPot Awareness Skill -->

## OpenPot Calendar Tab

You are connected to OpenPot, which has a **Calendar tab** — a native
Month and Agenda view that displays events you post to the calendar API.
The calendar is read-only for the user. You are the source of truth.

Events you create appear in the user's Calendar tab immediately. Use this
for anything time-bound: meetings, deadlines, reminders, cron job schedules,
maintenance windows, and recurring routines.

---

### Calendar API

**Endpoint:** `GET /api/calendar/events`
**Query params:** `start` and `end` (date strings, see format rules below)
**Auth:** Bearer token (same as other API endpoints on port 8000)

OpenPot fetches this endpoint to populate the Calendar tab. Your HTTP
server must accept `start` and `end` query parameters and return matching
events.

**Example request:**
```
GET /api/calendar/events?start=2026-04-01&end=2026-04-30
```

OpenPot may send either plain date strings (`2026-04-01`) or ISO 8601
timestamps (`2026-04-01T04:00:00Z`). Strip the `T` component if your
backend only handles plain dates:

```python
start_str = request.args.get("start", "")
start_date = start_str.split("T")[0]  # "2026-04-01T04:00:00Z" → "2026-04-01"
```

**Response shape:**

```json
{
  "events": [
    {
      "id": "google-cal-abc123",
      "title": "Weekly DCA Review",
      "start_date": "2026-04-14T09:00:00-04:00",
      "end_date": "2026-04-14T09:30:00-04:00",
      "is_all_day": false,
      "calendar_name": "Personal",
      "calendar_color": "#30D158",
      "source": "google_calendar",
      "notes": "Check DCA signals for CRCL, AFRM, TSLA",
      "location": null,
      "status": "confirmed",
      "created_by": null,
      "tags": ["finance"]
    },
    {
      "id": "reminder-oil-change-2026-04-20",
      "title": "Oil Change Due",
      "start_date": "2026-04-20",
      "is_all_day": true,
      "calendar_color": "#FF9500",
      "source": "reminder"
    }
  ]
}
```

---

### CRITICAL: Date Format Rules

OpenPot's date decoder handles two formats. Use the right one or events
will fail to decode silently.

| Event type | Format | Example |
|------------|--------|---------|
| Timed event | ISO 8601 with timezone offset | `"2026-04-14T09:00:00-04:00"` |
| All-day event | Date only, no time | `"2026-04-20"` |

**Rules:**
- Timed events MUST include a timezone offset (`-04:00`, `+00:00`, `Z`).
  UTC-only is fine: `"2026-04-14T13:00:00Z"`
- All-day events MUST use date-only format `"YYYY-MM-DD"` — do NOT
  include a time component for all-day events
- Fractional seconds are supported: `"2026-04-14T09:00:00.000-04:00"`
- Naive datetimes (no offset) are NOT supported — always include offset

---

### CRITICAL: id Field Must Be a String

The `id` field is a **string**, not a UUID object. Any stable string is
valid. Good patterns:

```python
# Google Calendar event ID (already a string)
"id": event["id"]

# Deterministic slug for cron jobs
"id": "cron-portfolio-review-2026-04-14"

# Prefixed UUID
"id": str(uuid.uuid4())
```

Do NOT use Python's `uuid.UUID` object directly — serialize it to a
string first. Do NOT use integer IDs — the field is typed as a string.

---

### Event Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | string | Yes | Stable unique ID. Never change it — OpenPot uses it to track state. |
| title | string | Yes | Short event title. Keep under 60 characters. |
| start_date | string | Yes | ISO 8601 with offset (timed) or `YYYY-MM-DD` (all-day). |
| end_date | string | No | ISO 8601 with offset. Omit for point-in-time or all-day events. |
| is_all_day | boolean | No | Defaults to false. Set true for date-only events. |
| calendar_name | string | No | Name of the source calendar (e.g. "Personal", "Work"). |
| calendar_color | string | No | Hex color for the leading color bar. Defaults to `#D4A843` if omitted. |
| source | string | No | See source enum below. Defaults to `agent`. |
| notes | string | No | Plain text only. No HTML — it will render as raw tags in the app. |
| location | string | No | Optional location string shown in Agenda view. |
| status | string | No | `"confirmed"`, `"tentative"`, or `"cancelled"`. Informational only. |
| created_by | string | No | Name of the agent or person who created the event. |
| tags | array of strings | No | Optional tags for filtering. Can be omitted entirely. |

---

### Source Enum Values

The `source` field tells OpenPot what icon to show and how to color
uncolored events. Always use the raw string value exactly as shown.

| Value | Icon | Use For |
|-------|------|---------|
| `google_calendar` | calendar | Events synced from Google Calendar |
| `agent_cron` | clock.arrow.2.circlepath | Recurring agent cron jobs |
| `agent` | person.crop.circle | One-off events scheduled by the agent |
| `agent_detected` | person.crop.circle.badge.clock | Agent inferred from conversation |
| `cron` | clock.arrow.2.circlepath | System cron jobs (non-agent) |
| `reminder` | bell | User-requested reminders |
| `system` | server.rack | Maintenance windows, server events |
| `manual` | pencil | User-created events the agent is tracking |
| `unknown` | questionmark | Fallback — avoid using |

---

### Calendar Colors

Use `calendar_color` to set the leading color bar in the event row.
These align with Pulse card categories:

| Color | Hex | Use For |
|-------|-----|---------|
| Agent gold | `#D4A843` | General agent-created events (default) |
| Finance green | `#30D158` | Financial reviews, DCA schedules |
| Calendar blue | `#378ADD` | Meetings, appointments, Google Calendar default |
| Health red | `#FF6B6B` | Medical appointments, medication schedules |
| System teal | `#4ECDC4` | Maintenance windows, server events |
| Projects dark teal | `#2AACAC` | Project deadlines, milestones |
| Reminder yellow | `#FFD60A` | User-requested reminders |
| Orange | `#FF9500` | Reminders, deadlines |

When syncing from Google Calendar, use the calendar's native
`backgroundColor` field directly — it's already a hex color.

---

### Calendar Views

Both views are populated from the same `/api/calendar/events` endpoint.

**Month view** — Grid calendar. Days with events show colored dot
indicators (up to 2 dots on iPhone, 3 on iPad). Tap a day to see its
events in a panel below the grid. Tap the "Show Events" bar to expand
cells and see event titles inline.

**Agenda view** — Chronological list from today through end of next
month. Events are grouped by day with sticky date headers. All-day events
appear first within each day. Each row shows: color bar, time (or
"all-day"), title, notes (truncated), and a trailing source icon.

---

### Backend Setup: Google Calendar Integration

To serve real Google Calendar events, your HTTP server needs Google API
credentials and a list endpoint that queries all calendars.

#### Install dependencies

```bash
pip install google-api-python-client google-auth-oauthlib google-auth-httplib2
```

#### Find your credentials

Your agent likely already has OAuth tokens from Google Calendar access.
Look in the workspace root or the API server directory:

```bash
ls ~/workspace/tokens.json
ls ~/workspace/gcp-oauth.keys.json
ls /opt/openbrain/tokens.json
```

If not present, ask the user:
- `tokens.json` — the OAuth token file (auto-refreshed)
- `gcp-oauth.keys.json` or `client_secrets.json` — the OAuth app credentials
  containing `client_id` and `client_secret`

#### Query all calendars (not just primary)

OpenPot displays events from ALL calendars. Do NOT query only the primary
calendar — use `calendarList().list()` to enumerate every calendar the
user has access to, then query each one.

```python
from googleapiclient.discovery import build
from google.oauth2.credentials import Credentials

def get_calendar_service():
    creds = Credentials.from_authorized_user_file("tokens.json")
    return build("calendar", "v3", credentials=creds)

def fetch_all_events(start_date: str, end_date: str) -> list:
    service = get_calendar_service()
    events = []

    # List every calendar the user has
    calendar_list = service.calendarList().list().execute()

    for cal in calendar_list.get("items", []):
        cal_id = cal["id"]
        cal_color = cal.get("backgroundColor", "#378ADD")
        cal_name = cal.get("summary", "")

        try:
            result = service.events().list(
                calendarId=cal_id,
                timeMin=start_date + "T00:00:00Z",
                timeMax=end_date + "T23:59:59Z",
                singleEvents=True,
                orderBy="startTime",
                maxResults=250,
            ).execute()

            for event in result.get("items", []):
                if event.get("status") == "cancelled":
                    continue
                start = event["start"].get("dateTime") or event["start"].get("date")
                end   = event["end"].get("dateTime")   or event["end"].get("date")
                is_all_day = "date" in event["start"] and "dateTime" not in event["start"]

                events.append({
                    "id": event["id"],
                    "title": event.get("summary", "(No title)"),
                    "start_date": start,
                    "end_date": end if not is_all_day else None,
                    "is_all_day": is_all_day,
                    "calendar_name": cal_name,
                    "calendar_color": cal_color,
                    "source": "google_calendar",
                    "notes": event.get("description"),
                    "location": event.get("location"),
                    "status": event.get("status"),
                    "tags": [],
                })
        except Exception as e:
            # Log and skip calendars that fail (e.g. holiday calendars)
            print(f"[Calendar] Skipping {cal_id}: {e}")
            continue

    return events
```

#### FastAPI endpoint

```python
from fastapi import APIRouter, Query
router = APIRouter()

@router.get("/api/calendar/events")
def get_calendar_events(start: str = Query(...), end: str = Query(...)):
    # Strip time component if caller sends ISO 8601 timestamp
    start_date = start.split("T")[0]
    end_date   = end.split("T")[0]
    events = fetch_all_events(start_date, end_date)
    return {"events": events}
```

#### Systemd service configuration

Your FastAPI server must have access to the OAuth token file and
credentials. Configure the service file to point to the right files:

```ini
[Unit]
Description=OpenBrain API
After=network.target

[Service]
User=vmocadmin
WorkingDirectory=/opt/openbrain
EnvironmentFile=/opt/openbrain/.env.service
ExecStart=/opt/openbrain/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000
Restart=always

[Install]
WantedBy=multi-user.target
```

**All environment variables must be in `.env.service`** — they do NOT
persist across reboots if set with `export` in a shell session.

```bash
# /opt/openbrain/.env.service
OPENBRAIN_API_TOKEN=sk-openbrain-prod-2026-03-26
POSTGRES_PASSWORD=your_postgres_password
GOOGLE_CLIENT_ID=your_client_id_from_gcp_console
GOOGLE_CLIENT_SECRET=your_client_secret_from_gcp_console
GOOGLE_TOKEN_FILE=/opt/openbrain/tokens.json
```

After modifying the service file or `.env.service`:

```bash
sudo systemctl daemon-reload
sudo systemctl restart openbrain-api.service
sudo systemctl status openbrain-api.service
```

---

### Calendar Authorization Rules

**The agent NEVER adds, modifies, or deletes calendar events without
explicit user permission.**

- **Google Calendar events** — always shown, never modified by the agent
  unless the user explicitly asks
- **Cron job events** — show in calendar only if the user has approved
  the cron job; ask before creating
- **Agent-created events** — always ask before creating: "Would you like
  me to add this to your OpenPot calendar?"
- **Reminders** — add to calendar only when the user says "remind me" or
  "add this to my calendar"
- Never infer that the user wants something on their calendar — always ask

When showing the calendar to the user, you can reference it freely. But
creating, updating, or deleting events requires a clear "yes" from the user.

---

### What to Put on the Calendar

**Post calendar events for:**
- Scheduled cron jobs the user cares about ("Portfolio review runs Monday 4:30 PM")
- Recurring routines the user is tracking ("Oil change due April 20")
- Deadlines the agent is managing ("Tax filing deadline April 15")
- Meetings the user mentioned ("Mark call — Friday 2 PM")
- Reminders the user explicitly asked the agent to track
- Google Calendar events (synced automatically when the endpoint is set up)

**Do NOT post:**
- Every internal agent operation — only events the user would care about
- Duplicate reminders that are already in Google Calendar
- Past events — keep the endpoint's result list pruned of stale entries
- HTML in the `notes` field — it renders as raw tags in the app

---

### Troubleshooting

**Calendar tab is empty:**
1. Check `GET /api/calendar/events?start=2026-04-01&end=2026-04-30` returns events
2. Confirm `openpot-status.json` has `"calendar": {"installed": true}`
3. Restart FastAPI after updating `openpot-status.json`:
   `sudo systemctl restart openbrain-api.service`
4. In OpenPot, pull to refresh on the Calendar tab

**Events decode silently — tab loads but shows no events:**
- Check that `id` is a string, not a UUID object
- Check that timed events have timezone offsets in `start_date`
- Check that all-day events use `"YYYY-MM-DD"` format, not `"YYYY-MM-DDT00:00:00"`
- Check the iOS console log for `[Calendar] Decode error:` entries

**Only showing primary calendar events:**
- Your endpoint is querying `calendarId="primary"` — change to iterate
  `calendarList().list()` (see backend setup above)

**Colors all showing as blue:**
- Set `calendar_color` on each event from the Google Calendar's
  `backgroundColor` field — the app uses this first before falling back
  to source-based colors

**"Not Authorized" or 401 on calendar endpoint:**
- Bearer token in the request must match `OPENBRAIN_API_TOKEN` in
  `.env.service`
- Confirm env vars are in the file, not just exported in a shell session

**Google OAuth error on server restart:**
- `tokens.json` path must be absolute, not relative to a working directory
- Set `GOOGLE_TOKEN_FILE` in `.env.service` with the full path

**Start/end parameter errors (400 or date parse failure):**
- OpenPot may send `2026-04-01T04:00:00Z` — strip the `T` component:
  `start_date = request.args.get("start").split("T")[0]`

---

### 9 Common Pitfalls

| # | Pitfall | Symptom | Fix |
|---|---------|---------|-----|
| 1 | Date format mismatch (time in all-day field) | All-day events fail to decode | Use `"YYYY-MM-DD"` for all-day events, no time component |
| 2 | UUID object instead of string id | All events fail to decode | Serialize with `str(event_id)` before returning |
| 3 | Missing OAuth credentials after deploy | 500 error from calendar endpoint | Add `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, token path to `.env.service` |
| 4 | Env vars lost on service restart | 500 after reboot, works in shell | Move all vars from `export` shell commands into `.env.service` |
| 5 | Only querying primary calendar | Missing work/family/hobby calendars | Iterate `calendarList().list()` to get all calendar IDs |
| 6 | `openpot-status.json` stale after setup | Calendar tab still shows empty | Update status file AND restart FastAPI (`systemctl restart openbrain-api`) |
| 7 | HTML in notes field | `<br>`, `<b>` tags visible in app | Strip HTML from `description` before returning: `re.sub(r'<[^>]+>', '', notes)` |
| 8 | All-day event with time component | All-day events show wrong time | Use Google's `event["start"]["date"]` not `event["start"]["dateTime"]` for all-day |
| 9 | Agent adds event without asking | User surprised by calendar changes | Always ask: "Would you like me to add this to your calendar?" |

<!-- END OPENPOT INSERT: calendar v2 -->
