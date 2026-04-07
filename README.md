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
4. The agent writes `openpot-status.json` to its workspace — OpenPot reads this on connect to know what features to show

When OpenPot ships new features, we update the manifest. Your agent picks up the changes on next sync.

## Install

### Option A: Manual

```bash
mkdir -p ~/.openclaw/workspace/skills/openpot-awareness
curl -o ~/.openclaw/workspace/skills/openpot-awareness/SKILL.md \
  https://raw.githubusercontent.com/AlMnotAi/openpot-agent/main/skill/SKILL.md
```

Restart the gateway. Then tell your agent: **"OpenPot sync"**

### Option B: ClawHub (when published)

```bash
openclaw plugins install openpot-awareness
```

## Current Features (manifest v1)

| Feature | Version | Needs HTTP Server | Description |
|---------|---------|-------------------|-------------|
| Chat | 1 | No | WebSocket chat via gateway |
| Pulse Cards | 1 | Yes | Proactive notification cards |
| Web Apps | 1 | Yes | Agent-built HTML apps in WKWebView |
| Skills Display | 1 | No | View/manage skills from the app |
| SSH Terminal | 1 | No | In-app terminal |

## Repo Structure

```
openpot-agent/
├── manifest.json              # Feature manifest (source of truth)
├── skill/
│   └── SKILL.md               # The awareness skill
├── inserts/
│   ├── pulse-cards-v1.md      # Pulse card behavioral directives
│   └── apps-v1.md             # Web apps behavioral directives
├── templates/
│   └── openpot-status-template.json
├── docs/
│   └── installation.md
└── README.md
```

## Adding a Feature

When a new feature ships in OpenPot:

1. Add it to `manifest.json`
2. Write an insert in `inserts/` (if agent behavior changes are needed)
3. Agents running the skill pick it up on next sync

## License

MIT

---

Built by [Al Menard](https://openpot.app). OpenPot is a native iOS app for self-hosted AI agents.
