---
name: openpot-awareness
description: Keeps this agent synced with OpenPot features — knows what the app can render and what content you need to produce
emoji: 🫕
version: 2.0.0
homepage: https://openpot.app/docs/awareness-skill
always: false
requirements:
  bins: []
  env: []
  config: []
---

# OpenPot Awareness Skill

You are connected to **OpenPot** — a native iOS app that serves as a
command center for AI agents. OpenPot has six tabs: **Chat**, **Pulse**
(notification cards), **Calendar**, **Apps**, **Terminal**, and **Agents**.
This skill keeps you aware of what OpenPot features exist and what you
need to do to support them.

## Source of Truth

The OpenPot Feature Manifest lives at:
`https://raw.githubusercontent.com/AlMnotAi/openpot-agent/main/manifest.json`

This JSON file lists every shipped OpenPot feature, what version it's
at, what endpoints it needs, and whether there's a behavioral insert
you should install.

Your local status is tracked in `openpot-status.json` in your workspace
root. OpenPot reads this file when it connects to know which features
you support.

---

## Triggers

Activate this skill when:

- The user says **"OpenPot sync"**, **"update OpenPot features"**,
  **"check OpenPot status"**, or **"what OpenPot features do I support?"**
- The user asks about a specific OpenPot feature and you're unsure of
  your current configuration
- The user asks for help setting up Calendar, Voice, Pulse, or any
  other OpenPot feature
- The user reports a broken or missing feature in OpenPot
- During a **heartbeat or nightly maintenance cycle** (optional)

---

## Behavior: Full Sync

### Step 1 — Fetch the manifest

Read the file at:
```
https://raw.githubusercontent.com/AlMnotAi/openpot-agent/main/manifest.json
```

Use your web search or browser tool to fetch it. Parse it as JSON.
If the fetch fails, report the failure and continue with cached status.

### Step 2 — Read your current status

Read `openpot-status.json` from your workspace root. If the file does
not exist, this is a first-time setup — treat all features as not
installed.

### Step 3 — Compare and report

For each feature in the manifest:

- **Already installed at current version** → skip, report as ✅
- **Not installed** → check prerequisites:
  - `gateway: true` — you have this if you're running on OpenClaw
  - `http_server: true` — do you serve HTTP on port 8000?
  - `endpoints` — do these routes exist on your server?
  - `python_packages` — are these packages installed in your venv?
- If prerequisites met → offer to install
- If prerequisites NOT met → report what's missing
- **Installed but older version** → update available, offer to install

### Step 4 — Install (with user confirmation)

For features that need an insert:

1. Fetch the insert file from the repo (URL in `insert_file` field)
2. Save a copy to `openpot-inserts/` in your workspace
3. Show the user what will be added to SOUL.md
4. **Ask for confirmation** before modifying SOUL.md
5. On confirmation: append the insert to SOUL.md
6. Update `openpot-status.json`
7. **Restart FastAPI** after updating `openpot-status.json`:
   `sudo systemctl restart openbrain-api.service`
   OpenPot reads the status on connect — a stale cached response will
   hide newly-enabled features until the service restarts.

### Step 5 — Write status

Write the updated `openpot-status.json` to your workspace root.

---

## Behavior: Status Check

When the user asks what features they support:

1. Read `openpot-status.json`
2. Present a clean summary:

```
OpenPot Feature Status
━━━━━━━━━━━━━━━━━━━━━
✅ Chat (v1)
✅ Pulse Cards (v1)
✅ Pulse Expansion (v1)
✅ Calendar (v2)
✅ Voice (v1)
✅ Web Apps (v1)
✅ Skills Display (v1)
✅ SSH Terminal (v1)

Last synced: 2026-04-09 08:00 ET
```

---

## Behavior: Voice Setup Assistance

When the user asks about setting up voice, or says "I want to use voice
with OpenPot", or "set up ElevenLabs":

1. Clarify that voice is entirely configured in the OpenPot app — no
   server changes required
2. Walk through: Settings → Voice Output → ElevenLabs → enter API key
3. Offer to help find a Voice ID from their ElevenLabs account
4. Mention per-agent voice: Agents tab → tap agent → Voice section
5. Warn about Talk Mode: do NOT add `operator.talk.secrets` scope —
   it breaks message delivery

After voice is configured, update `openpot-status.json` to set
`"voice": {"installed": true, "version": 1}`.

---

## Behavior: Calendar Setup Assistance

When the user asks about setting up the Calendar tab, or says "my
calendar is empty", or "set up calendar":

1. Check if `GET /api/calendar/events` exists on their HTTP server
2. Check if `openpot-status.json` has `"calendar": {"installed": true}`
3. Check if Google OAuth credentials (`tokens.json`) are accessible
4. Walk through the backend setup in the calendar-v2 insert
5. Remind the user: after updating `openpot-status.json`, restart
   FastAPI with `sudo systemctl restart openbrain-api.service`

**Calendar authorization rules to follow:**
- Never add, modify, or delete calendar events without explicit user
  permission
- Google Calendar events are always shown, never modified by the agent
  unless the user explicitly asks
- Always ask before creating agent-created events: "Would you like me
  to add this to your OpenPot calendar?"

---

## Behavior: Troubleshooting

When the user reports a broken OpenPot feature, work through this
checklist before suggesting more complex fixes:

1. **Is `openpot-status.json` up to date?** Read it. Does it reflect
   the feature as installed?
2. **Did FastAPI restart after the last status change?**
   `sudo systemctl status openbrain-api.service` — check the start time
3. **Is the endpoint reachable?** `curl -H "Authorization: Bearer TOKEN"
   http://localhost:8000/api/openpot/status`
4. **Are env vars in `.env.service`?** Not just exported in a shell
   session — they must be in the file to survive reboots
5. **For calendar:** Are events using string IDs (not UUID objects)?
   Are timed events using ISO 8601 with timezone offset?
6. **For Pulse:** Is the HTTP server running on port 8000? Do the
   `/api/cards` endpoints exist?

Report findings clearly, then suggest the specific fix.

---

## Behavior: Nightly Auto-Check (Optional)

If you run a nightly maintenance cycle:

1. Fetch the manifest silently
2. Compare against `openpot-status.json`
3. If updates available, note in MEMORY.md:
   `"OpenPot updates available: [list]. Run 'OpenPot sync' to install."`
4. **Do NOT auto-modify SOUL.md** without user confirmation
5. **Do** silently update timestamps in `openpot-status.json`

---

## Insert Management

Insert blocks in SOUL.md are bounded by HTML comments:

```
<!-- OPENPOT INSERT: feature-id vN — installed by openpot-awareness skill -->
...content...
<!-- END OPENPOT INSERT: feature-id vN -->
```

- **Fresh install:** Append insert block to end of SOUL.md
- **Upgrade (e.g. calendar v1 → v2):** Find old block by opening
  comment tag, remove the entire block, append the new version
- **Never edit insert content manually** — updates come from the repo
- **SOUL.md capacity warning:** If SOUL.md is approaching 90% of your
  context limit, warn the user before appending a new insert. Large
  inserts can push your effective context over the limit.

---

## Rules

- **NEVER auto-modify SOUL.md** without explicit user confirmation
- **ALWAYS save fetched inserts** to `openpot-inserts/` before applying
- **ALWAYS update `openpot-status.json`** after any change
- **ALWAYS restart FastAPI** after updating `openpot-status.json`
- If manifest fetch fails, use cached status and report the failure
- If a feature has `depends_on` and the dependency isn't installed,
  install the dependency first (with user confirmation)
- Voice needs no insert and no server changes — mark it installed after
  the user confirms they've entered credentials in the app
