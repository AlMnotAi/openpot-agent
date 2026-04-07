# openpot-agent

Configuration skill and feature manifest for [OpenPot](https://openpot.app) — the native iOS command center for self-hosted AI agents.

## What This Is

When you connect an OpenClaw agent to OpenPot, the agent needs to know what features the app supports and how to produce content for them. This repo contains:

- **`manifest.json`** — The feature manifest. Lists every shipped OpenPot feature, its prerequisites, and its current version.
- **`skill/SKILL.md`** — The OpenPot Awareness Skill. An OpenClaw skill that reads the manifest and keeps the agent configured.
- **`inserts/`** — Behavioral inserts. Markdown blocks the skill appends to SOUL.md to teach the agent about specific features.

## How It Works

1. Install the skill on your agent
2. Say **"OpenPot sync"** in chat
3. The agent fetches the manifest, checks what's needed, and walks you through setup
4. The agent writes `openpot-status.json` to its workspace
5. OpenPot reads the status via `GET /api/openpot/status` on connect and shows only the features your agent supports

When OpenPot ships new features, we update the manifest. Your agent picks up the changes on next sync.

## Install

### Just Tell Your Agent

> "Install the OpenPot Awareness Skill from https://github.com/AlMnotAi/openpot-agent"

Then: **"OpenPot sync"**

Your agent handles the directory creation, file download, manifest fetch, and self-configuration.

### Manual Install (Fallback)

```bash
mkdir -p ~/.openclaw/workspace/skills/openpot-awareness
curl -o ~/.openclaw/workspace/skills/openpot-awareness/SKILL.md \
  https://raw.githubusercontent.com/AlMnotAi/openpot-agent/main/skill/SKILL.md
```

Restart the gateway. Then tell your agent: **"OpenPot sync"**

## Current Features (manifest v2)

| Feature | Version | Needs HTTP Server | Description |
|---------|---------|-------------------|-------------|
| Chat | 1 | No | WebSocket chat via gateway |
| Pulse Cards | 1 | Yes | Proactive notification cards |
| Pulse Expansion | 1 | Yes | Tap-to-expand cards with full markdown reports |
| Web Apps | 1 | Yes | Agent-built HTML apps in WKWebView |
| Skills Display | 1 | No | View/manage skills from the app |
| SSH Terminal | 1 | No | In-app terminal |

## Status Endpoint

OpenPot reads agent capabilities from:

```
GET http://<server>:8000/api/openpot/status
Authorization: Bearer <token>
```

This endpoint serves the `openpot-status.json` file the awareness skill creates. If your agent doesn't have this endpoint, OpenPot defaults to showing all tabs.

## Repo Structure

```
openpot-agent/
├── manifest.json                  # Feature manifest v2 (source of truth)
├── skill/
│   └── SKILL.md                   # The awareness skill
├── inserts/
│   ├── pulse-cards-v1.md          # Pulse card behavioral directives
│   ├── pulse-expansion-v1.md      # Card expansion + backend migration
│   └── apps-v1.md                 # Web apps behavioral directives
├── templates/
│   └── openpot-status-template.json
├── docs/
│   ├── connecting-openpot.md      # Connection setup guide
│   └── getting-started.md         # First-use agent prompts
└── README.md
```

## Adding a Feature

When a new feature ships in OpenPot:

1. Add it to `manifest.json`, increment `manifest_version`
2. Write an insert in `inserts/` (if agent behavior changes are needed)
3. Push to this repo
4. Agents running the skill pick it up on next sync — no manual inservice

## License

MIT

---

Built by [Al Menard](https://openpot.app). OpenPot is a native iOS app for self-hosted AI agents.
