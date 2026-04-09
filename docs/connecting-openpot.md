# Connecting OpenPot to Your Agent

## What You Need

- OpenPot installed on your iPhone or iPad
- An OpenClaw agent running on your server (Linux VM or Mac)
- Your server's IP address (LAN and/or Tailscale)
- Your gateway token

---

## Step 1: Find Your Gateway Address

Your agent's gateway runs on port **18789**. The address you use depends on where you are.

### When you're on the same WiFi as your server (at home)

Use your server's local IP address.

**To find it, ask your agent:** "What's your local IP address?"

Or on the server directly:

- Linux: `hostname -I` (first address listed)
- Mac: `ipconfig getifaddr en0`

Your gateway address looks like: `ws://192.168.1.XXX:18789`

### When you're away from home (cellular, other WiFi)

You need **Tailscale** installed on both your server and your phone. Tailscale creates a private network between your devices that works anywhere.

**To find your Tailscale IP, ask your agent:** "What's your Tailscale IP?"

Or on the server directly: `tailscale ip -4`

Your gateway address looks like: `ws://100.XXX.XXX.XXX:18789`

**If you don't have Tailscale set up yet:** OpenPot will only work when you're on the same WiFi as your server. Set up Tailscale later to get remote access. Your agent can help — just ask it to install and configure Tailscale.

---

## Step 2: Find Your Gateway Token

This was shown during OpenClaw's onboarding wizard when you first set up your agent. If you saved it, use it.

**If you don't have it, ask your agent:** "What's my gateway token?"

Or on the server directly: `openclaw config get gateway.token`

---

## Step 3: Add Your Agent in OpenPot

1. Open OpenPot
2. Go to the **Agents** tab
3. Tap **Add Agent**
4. Enter a name for your agent (whatever you call it — "Larry", "Claire", "Home Agent")
5. Enter the gateway URL: `ws://YOUR_IP_ADDRESS:18789`
6. Enter the gateway token
7. Save

---

## Step 4: Approve the Connection

The first time OpenPot connects, the gateway needs to verify your phone is allowed to connect. This is a security feature — it prevents unauthorized devices from talking to your agent.

**Your agent will see the pairing request.** Tell your agent in any channel (Telegram, Slack, or any chat):

> "Approve the OpenPot device"

Your agent approves it and OpenPot connects. This is a one-time step — after approval, OpenPot connects automatically every time you open the app.

---

## Setting Up Both LAN and Tailscale (Recommended)

For the best experience, add **two connection routes** for your agent. OpenPot tries them in priority order and uses whichever one works.

In the Agents tab, tap your agent → **Edit** → add both routes:

| Route | Address | Priority | When It's Used |
|-------|---------|----------|----------------|
| LAN | `ws://192.168.1.XXX:18789` | 1 (primary) | You're on home WiFi — faster |
| Tailscale | `ws://100.XXX.XXX.XXX:18789` | 2 (fallback) | You're away from home |

OpenPot tries LAN first. If it can't reach it (you're not home), it falls back to Tailscale automatically. You don't switch anything manually — the app handles it.

---

## Installing the OpenPot Awareness Skill

Once connected, tell your agent:

> "Install the OpenPot Awareness Skill from https://github.com/AlMnotAi/openpot-agent"

Then:

> "OpenPot sync"

Your agent installs the skill, checks what features it can support, and configures itself. It writes an `openpot-status.json` file that OpenPot reads on connect to know what features to show.

---

## How OpenPot Reads Your Agent's Capabilities

When OpenPot connects to your agent, it reads the agent's feature status from:

```
GET http://<your-server>:8000/api/openpot/status
```

This endpoint serves the `openpot-status.json` file that the awareness skill creates. It tells OpenPot which features your agent supports — Pulse cards, web apps, calendar, voice, terminal, and any future features.

If your agent doesn't have the awareness skill installed yet, or doesn't have an HTTP server on port 8000, OpenPot defaults to showing all tabs. Nothing breaks — the status check is optional.

**Important:** After updating `openpot-status.json`, restart your FastAPI service so OpenPot receives the new status on next connect:

```bash
sudo systemctl restart openbrain-api.service
```

---

## What Each Feature Requires

| Feature | Requires | What You Get |
|---------|----------|--------------|
| Chat | Gateway only | Real-time chat, streaming responses, message history |
| Pulse Cards | Gateway + HTTP server + card endpoints | Proactive notification cards |
| Pulse Expansion | Pulse Cards | Tap-to-expand report cards with full markdown |
| Calendar | Gateway + HTTP server + `/api/calendar/events` | Month and Agenda views with Google Calendar sync |
| Voice | Gateway only (user enters credentials in app) | Mic input + ElevenLabs or System Voice TTS output |
| Web Apps | Gateway + HTTP server + app endpoints | In-app browser for agent-built HTML tools |
| Skills | Gateway only | View installed and available skills (in Agents tab) |
| Terminal | Gateway only | SSH connection to your server |

Voice requires no server setup — the user enters ElevenLabs credentials in OpenPot Settings on each device. Each device that uses OpenPot needs its own credentials entered separately.

The Calendar tab requires your HTTP server to implement `GET /api/calendar/events`. If the endpoint isn't available, the Calendar tab shows empty. Tell your agent to set it up:

> "Set up the calendar API endpoint so OpenPot can display my calendar"

---

## Troubleshooting

**"Connecting..." stays amber and never turns green:**
- Check that your address starts with `ws://` not `http://`
- Check that port is `18789`
- If using Tailscale, make sure Tailscale is running on your phone (check the Tailscale app)
- Ask your agent: "Is the gateway running?"

**Works on WiFi but not on cellular:**
- You need Tailscale. The LAN address (192.168.x.x) only works on your home network.
- Make sure Tailscale is installed on both your server and your phone

**"Device not approved" or connection rejected:**
- Tell your agent to approve the device: "Approve the OpenPot device"
- If your agent can't do it, on the server run: `openclaw devices approve`

**Connected but Pulse tab shows "No Pulse Server":**
- Your agent needs an HTTP server on port 8000 with card endpoints
- Run `curl http://YOUR_SERVER_IP:8000/api/cards` to verify the endpoint exists
- If the endpoint returns 500: check that env vars (OPENBRAIN_API_TOKEN, POSTGRES_PASSWORD) are in `/opt/openbrain/.env.service` — not just exported in a shell session. Variables set with `export` in a shell do not survive service restarts.
- After fixing the env file: `sudo systemctl daemon-reload && sudo systemctl restart openbrain-api.service`
- Check `openpot-status.json` has `"pulse_cards": {"enabled": true}`
- Restart FastAPI after any status file change

**Calendar tab is empty:**
- Your agent needs `GET /api/calendar/events` implemented on its HTTP server
- Tell your agent: "OpenPot sync" — it will check what's needed and set it up
- If you had calendar working before and it stopped: check that env vars survived the last server reboot (see Pulse 500 fix above — same root cause)
- See also: the 9 calendar pitfalls in the calendar-v2 insert (fetch via OpenPot sync)

**Calendar shows events but wrong colors:**
- Ensure `calendar_color` is set on each event from the Google Calendar's native `backgroundColor`
- The app uses `calendar_color` first; if missing it falls back to source-based defaults

**Voice mic button does nothing:**
- iOS speech recognition permission may be denied: Settings → Privacy & Security → Speech Recognition → enable OpenPot
- Speak more clearly and closer to the mic

**ElevenLabs voice not playing:**
- API key not set or expired: Settings → Voice Output → re-enter API key
- Check per-agent Voice ID is set in Agents tab → tap agent → Voice section

**Agent name shows as blank:**
- Your agent needs IDENTITY.md configured in its workspace with a name and emoji
