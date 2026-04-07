# Installing the OpenPot Awareness Skill

## Prerequisites

- An OpenClaw agent (any version with skill support)
- OpenPot iOS app connected to the agent's gateway

## Install the Skill

Copy the skill file to your agent's workspace:

```bash
mkdir -p ~/.openclaw/workspace/skills/openpot-awareness
curl -o ~/.openclaw/workspace/skills/openpot-awareness/SKILL.md \
  https://raw.githubusercontent.com/AlMnotAi/openpot-agent/main/skill/SKILL.md
```

If your workspace is at a different path (e.g., `~/.openclaw/workspace-prod/`), adjust accordingly.

Restart the gateway so the skill is loaded:

```bash
# Linux
systemctl --user restart openclaw-gateway

# macOS
openclaw gateway restart
```

## Run Your First Sync

Open a chat with your agent (via OpenPot, Telegram, Slack, or any channel) and say:

```
OpenPot sync
```

The agent will:
1. Fetch the feature manifest from GitHub
2. Check what prerequisites you have (gateway, HTTP server, etc.)
3. Report which features you can enable
4. Walk you through installing behavioral inserts for each feature
5. Write `openpot-status.json` to your workspace

## What Gets Modified

- **SOUL.md** — Feature inserts are appended (with your confirmation)
- **openpot-status.json** — Created/updated in workspace root (OpenPot reads this)
- **openpot-inserts/** — Directory created in workspace to cache insert files

## Verify

After syncing, say:

```
What OpenPot features do I support?
```

The agent will read `openpot-status.json` and report your feature status.

## For Agents Without an HTTP Server

If your agent only has the gateway (no FastAPI on port 8000), the sync will report:

- ✅ Chat — works with gateway only
- ✅ Skills Display — works with gateway only
- ✅ SSH Terminal — works with gateway only
- ❌ Pulse Cards — needs HTTP server with /api/cards endpoints
- ❌ Web Apps — needs HTTP server with /api/apps endpoints

You can still use Chat, Skills, and Terminal. Pulse Cards and Apps require setting up an HTTP server with the listed endpoints.
