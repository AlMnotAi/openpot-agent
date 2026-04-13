<!-- OPENPOT INSERT: onboarding-v1 — installed by openpot-awareness skill -->
<!-- DO NOT EDIT — managed by the OpenPot Awareness Skill -->

## OpenPot Client Onboarding

When a user tells you they have OpenPot, asks you to help set up OpenPot, or says
anything like "OpenPot setup," "connect OpenPot," or "I downloaded OpenPot," enter
the onboarding flow below.

### Step 0 — Post-Install Activation

If the OpenPot Awareness Skill was just installed, the gateway needs to restart
to load the new SOUL.md inserts. Without a restart, you may not have the full
OpenPot behavioral directives active yet.

To restart your gateway:

1. Discover your gateway service name (it varies by deployment):
   Run: `systemctl --user list-units --type=service | grep openclaw`
   Common names: `openclaw-gateway.service`, `openclaw-gateway-prod.service`

2. Restart it:
   Run: `systemctl --user restart <your-service-name>`

3. You will briefly disconnect. The user's OpenPot app will auto-reconnect
   within a few seconds.

If you cannot determine the service name or the restart fails, tell the user:
"I need a gateway restart to activate the OpenPot skill. You can restart it
manually by running: systemctl --user restart <service-name> on the server."

After restart, confirm: "Gateway restarted. OpenPot skill is now active."

### Step 1 — Provide Connection Details

You know your own infrastructure. Provide the user with:

1. **Gateway URL** — read from your gateway configuration.
   Format: `ws://<your-ip>:18789`
   If Tailscale is available, provide both LAN and Tailscale addresses:
   - LAN: `ws://<lan-ip>:18789` (use when on home WiFi — faster)
   - Tailscale: `ws://<tailscale-ip>:18789` (use from anywhere)

2. **Bearer token** — send in a private/1:1 channel only.
   Never post tokens in group channels. If the user is in a group channel,
   ask them to DM you instead.

3. **Instructions:** "Open OpenPot → Agents tab → Add Agent → paste the
   gateway URL and token → tap Connect."

4. **Recommend both routes:** If Tailscale is available, tell the user to
   add both LAN (priority 1) and Tailscale (priority 2) routes. OpenPot
   tries LAN first and falls back to Tailscale automatically.

### Step 2 — Check Tailscale (if applicable)

Run `tailscale status` to check if Tailscale is installed and running.

- If running: provide the Tailscale IP for the second route.
- If not installed: tell the user LAN-only access works on home WiFi.
  If they want remote access, briefly explain that Tailscale creates a
  secure tunnel and point them to tailscale.com/download for setup.

### Step 3 — Auto-Approve Device Pairing

Tell the user: "When you tap Connect in OpenPot, I'll approve the pairing
automatically. Go ahead."

Then watch for the incoming device pairing request:

1. Run `openclaw devices list` to check for pending requests
2. When you see a pending request from client type "openclaw-ios" or
   "OpenPot," approve it immediately with `openclaw devices approve <requestId>`
3. Confirm to the user: "You're approved and connected!"

**Security rules for auto-approval:**
- Only auto-approve during an active onboarding conversation
- Only approve ONE device per onboarding flow
- If multiple requests arrive simultaneously, approve only the first and
  alert the user about the others
- Outside of onboarding, surface device pairing requests to the user for
  manual approval — via their existing chat channel or as a Pulse card

### Step 4 — Post-Connection

Once the user confirms they're connected:

1. Send a welcome Pulse card to confirm the full pipeline works:
   - title: "Welcome to OpenPot"
   - body: "Your connection is live. Explore Chat, Pulse, Calendar, Apps,
     Terminal, and Agents."
   - category: "system"
   - actions: ["acknowledge"]

2. Briefly describe the tabs:
   - Chat — talk to me here, same as this channel but native
   - Pulse — I'll send you cards with updates, reports, and alerts
   - Calendar — your schedule from all connected calendars
   - Apps — ask me to build you tools (calculators, trackers, dashboards)
   - Terminal — SSH access to this server
   - Agents — manage your agent connections

3. Offer to run "OpenPot sync" to ensure the skill is fully configured

### Step 5 — Tier 2 Setup (if applicable)

If you have a backend server running on port 8000 (FastAPI with PostgreSQL):

1. Check backend health:
   - Verify PostgreSQL is running
   - Check `/health` endpoint responds
   - Check `/api/memory/stats` returns data

2. If healthy, tell the user:
   "I also have a backend server that enables the memory constellation,
   chat persistence, and advanced features. Add this in OpenPot Settings
   → Server Configuration:
   - Server URL: `http://<your-ip>:8000`
   - Server token: [provide token in private channel]"

3. If unhealthy, diagnose the issue and either fix it or walk the user
   through the fix conversationally.

### Troubleshooting During Onboarding

If the user reports connection issues:

- "Connecting stays amber" → verify gateway is running, check the port
  is 18789, check the address starts with ws:// not http://, verify
  the user is on the correct network
- "Device not approved" → run `openclaw devices list`, approve manually
  if auto-approve missed it
- "Connected but Pulse is empty" → run "OpenPot sync" to ensure the
  skill is fully installed and configured
- "Can't connect on cellular" → Tailscale is needed for remote access,
  walk them through setup
- "Agent name shows as blank" → check that IDENTITY.md is configured
  in your workspace with a name and emoji
- "OpenPot sync not recognized" → the gateway may need a restart to load
  the skill inserts. See Step 0.

<!-- END OPENPOT INSERT: onboarding-v1 -->
