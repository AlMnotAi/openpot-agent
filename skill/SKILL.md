---
name: openpot-awareness
description: Keeps this agent synced with OpenPot features — knows what the app can render and what content you need to produce
emoji: 🫕
version: 1.0.0
homepage: https://openpot.app/docs/awareness-skill
always: false
requirements:
  bins: []
  env: []
  config: []
---

# OpenPot Awareness Skill

You are connected to **OpenPot** — a native iOS app that serves as a
command center for AI agents. OpenPot has tabs for Chat, Pulse (cards),
Apps, Skills, Terminal, Files, and Agents. This skill keeps you aware
of what OpenPot features exist and what you need to do to support them.

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
✅ Web Apps (v1)
✅ Skills Display (v1)
✅ SSH Terminal (v1)

Last synced: 2026-04-07 08:00 ET
```

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
- **Upgrade:** Find old block by opening comment, remove it, append new version
- **Never edit insert content manually** — updates come from the repo

---

## Rules

- **NEVER auto-modify SOUL.md** without explicit user confirmation
- **ALWAYS save fetched inserts** to `openpot-inserts/` before applying
- **ALWAYS update `openpot-status.json`** after any change
- If manifest fetch fails, use cached status and report the failure
- If a feature has `depends_on` and the dependency isn't installed,
  install the dependency first (with user confirmation)
