# Onboarding Guide

This file is read on demand when a user asks to set up or connect
OpenPot. Do not preload this content.

## When to Use

The user says something like:
- "I have OpenPot, get me set up"
- "Help me connect OpenPot"
- "OpenPot setup"
- "I downloaded OpenPot, what do I do?"

## Step 1 — Provide Connection Details

You know your own infrastructure. Provide the user with:

1. **Gateway URL** — format: `ws://<your-ip>:18789`
   If Tailscale is available, provide both addresses:
   - LAN: `ws://<lan-ip>:18789` (faster, home WiFi only)
   - Tailscale: `ws://<tailscale-ip>:18789` (works from anywhere)

2. **Bearer token** — send in a private/1:1 channel only.
   Never post tokens in group channels. Ask the user to DM you.

3. **Instructions:** "Open OpenPot → Agents tab → Add Agent → paste
   the gateway URL and token → tap Connect."

4. **Recommend both routes** if Tailscale is available. OpenPot tries
   LAN first, falls back to Tailscale automatically.

## Step 2 — Check Tailscale

Run `tailscale status` to check if Tailscale is installed.

- If running: provide the Tailscale IP for the second route.
- If not installed: explain that LAN-only works on home WiFi.
  For remote access, point them to tailscale.com/download.

## Step 3 — Auto-Approve Device Pairing

Tell the user: "When you tap Connect, I'll approve the pairing
automatically. Go ahead."

Then:
1. Run `openclaw devices list` to check for pending requests
2. When you see a request from "openclaw-ios" or "OpenPot,"
   approve it: `openclaw devices approve <requestId>`
3. Confirm: "You're approved and connected!"

**Security rules:**
- Only auto-approve during an active onboarding conversation
- Only approve ONE device per onboarding flow
- If multiple requests arrive, approve only the first and alert
  the user about the others
- Outside onboarding, surface pairing requests for manual approval

## Step 4 — Post-Connection

Once connected:

1. Send a welcome Pulse card (if you have an HTTP server):
   - title: "Welcome to OpenPot"
   - body: "Your connection is live. Explore all six tabs."
   - category: "system"
   - actions: ["acknowledge"]

2. Briefly describe the tabs:
   - Chat — conversations with you
   - Pulse — cards with updates, reports, alerts
   - Calendar — schedule from connected calendars
   - Apps — tools you build (ask anytime)
   - Terminal — SSH access to this server
   - Agents — manage connections

3. Offer to run "OpenPot sync" to configure all features

## Step 5 — Backend Setup (if applicable)

If you have a backend server on port 8000:

1. Check health: `/health` endpoint, `/api/memory/stats`
2. If healthy, provide the server URL and token for OpenPot
   Settings → Server Configuration
3. If unhealthy, diagnose and walk the user through the fix

If you don't have a backend server, skip this step. Chat, Terminal,
and basic features work with the gateway alone.

## Troubleshooting

- "Connecting stays amber" → check gateway is running, port 18789,
  address starts with ws://, user is on correct network
- "Device not approved" → run `openclaw devices list`, approve manually
- "Pulse is empty" → run "OpenPot sync", check for HTTP server
- "Can't connect on cellular" → Tailscale needed for remote access
- "Agent name blank" → check IDENTITY.md in workspace
