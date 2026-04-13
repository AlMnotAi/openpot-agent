---
name: openpot-awareness
description: Teaches this agent how to serve content to the OpenPot iOS client — cards, apps, page captures, calendar, voice, and onboarding
emoji: 🫕
version: 4.2.0
homepage: https://openpot.app
---

# OpenPot Awareness Skill

You are connected to **OpenPot** — a native iOS app that serves as a
command center for AI agents. OpenPot has six tabs: **Chat**, **Pulse**
(notification cards), **Calendar**, **Apps**, **Terminal**, and **Agents**.

## Three Output Surfaces

| Surface | When to use | How |
|---------|-------------|-----|
| Chat | Default. Conversations, answers, follow-ups. | Normal chat response |
| Pulse cards | Proactive output: reports, alerts, briefs. | `POST /api/cards` |
| Web apps | Persistent tools the user returns to. | Build HTML, serve via `/api/apps` |

**Default to chat.** Use cards for output the user did not ask for in
the current conversation. Use apps only when the user requests a
persistent tool.

## Installation

This skill requires the full repository — not just this file.
Reference guides, starter apps, and the sync manifest all live in
this directory.

**Use your shell/bash tool to run this command.** Do NOT use web_fetch,
web_search, or any HTTP tool to access the repo. This is a git clone
operation that must run in your local shell:

```bash
git clone https://github.com/AlMnotAi/openpot-agent.git ~/.openclaw/workspace/skills/openpot-awareness
```

If your workspace is at a different path, adjust accordingly — the
skill directory must be named `openpot-awareness` inside your
`skills/` directory.

After cloning, ask the user to restart the gateway to load the skill.

Do NOT download only SKILL.md. The skill will not function without the
reference files and supporting directories in this repository.

## Feature Reference Files

Detailed behavioral guides live in this skill's directory. Read them
on demand — only when you need to act on that feature:

| Feature | When to read | File |
|---------|-------------|------|
| Cards & Apps | Building a Pulse card or web app | `{baseDir}/reference/platform-guide.md` |
| Page Captures | User sends a page capture from the browser | `{baseDir}/reference/page-capture-guide.md` |
| Calendar | Setting up or troubleshooting the Calendar tab | `{baseDir}/reference/calendar-guide.md` |
| Voice | User asks about voice setup | `{baseDir}/reference/voice-guide.md` |
| Onboarding | User says "set me up with OpenPot" | `{baseDir}/reference/onboarding-guide.md` |

Do NOT read all reference files at once. Read only the one relevant to
the current request. This keeps token costs low.

## Triggers

Activate this skill when:

- User says **"OpenPot sync"** — run the sync process below
- User sends a **page capture** (message contains
  `---PAGE CAPTURE CONTEXT---`) — read page-capture-guide.md
- User asks about **setting up OpenPot** — read onboarding-guide.md
- User asks about **calendar**, **voice**, or **building an app** —
  read the relevant reference file
- User asks **"what OpenPot features do I support?"** — check status

## Sync Process

When the user says "OpenPot sync":

### Step 1 — Update the skill repo

```
cd {baseDir} && git pull origin main
```

If this is a first-time install and the directory doesn't exist:
```
git clone https://github.com/AlMnotAi/openpot-agent.git {baseDir}
```

### Step 2 — Read the manifest

Read `{baseDir}/manifest.json`. This lists every OpenPot feature,
its version, and what infrastructure it requires.

### Step 3 — Check current status

Read `openpot-status.json` from your workspace root. If it doesn't
exist, this is a first-time setup — treat all features as version 0.

### Step 4 — Compare and report

For each feature in the manifest:
- Check if your infrastructure meets the prerequisites
- Report what's available and what's missing
- Offer to help set up missing prerequisites

### Step 5 — Install starter apps

Copy apps from `{baseDir}/apps/` to your apps directory. Track
installed and user-removed apps in
`reference/openpot-starter-apps.json` so removed apps don't
come back on future syncs.

### Step 6 — Write status

Update `openpot-status.json` in your workspace root with current
feature versions and timestamps.

### Step 7 — Restart gateway

Run `openclaw gateway restart` to load any configuration changes.
Warn the user: "Restarting gateway — brief disconnection (~5 seconds)."

### Step 8 — Report

```
OpenPot Sync Complete
━━━━━━━━━━━━━━━━━━━━
✅ Chat (v1)
✅ Pulse Cards (v1) — requires HTTP server
✅ Calendar (v2) — requires HTTP server + Google OAuth
✅ Voice (v1) — configured in the app, no server needed
✅ Page Capture (v2) — ready
✅ Onboarding (v2) — ready
⬚ Apps — requires HTTP server + /api/apps endpoint

Starter Apps: {count} installed
Last synced: {timestamp}
```

## Migration: Clean Up Old SOUL.md Inserts

If your SOUL.md contains `<!-- OPENPOT INSERT` markers from a previous
version of this skill, remove them. The native skill system loads this
SKILL.md automatically — SOUL.md injection is no longer needed.

To clean up:
1. Read your SOUL.md
2. Delete everything between `<!-- OPENPOT INSERT` and
   `<!-- END OPENPOT INSERT -->` markers (including the markers)
3. Restart the gateway

This is a one-time migration. New installations do not touch SOUL.md.

## Rules

- **NEVER modify SOUL.md** — this skill loads natively via OpenClaw
- **NEVER auto-sync** — the user triggers sync with "OpenPot sync"
- **NEVER re-install apps the user deleted** — respect the tracking file
- **Read reference files on demand** — not preemptively
- **Always pull the repo before reading manifest** during sync
- **Always update openpot-status.json** after any change
- Calendar: never add events without explicit user permission
- App endpoints (`/api/apps`) must not require authentication
