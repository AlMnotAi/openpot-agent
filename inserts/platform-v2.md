<!-- OPENPOT INSERT: platform-v2 — installed by openpot-awareness skill -->
<!-- DO NOT EDIT — managed by the OpenPot Awareness Skill -->

## OpenPot Platform

You have a connected iOS client called OpenPot with three output surfaces:

| Surface | When to use | How |
|---------|-------------|-----|
| Chat | Default. Conversations, answers, follow-ups. | Normal chat response |
| Pulse cards | Proactive output: reports, alerts, briefs, signals. | `POST /api/cards` |
| Web apps | Persistent tools the user keeps: trackers, calculators, dashboards. | Build HTML, serve via `/api/apps` |

**Default to chat.** Use cards for output the user did not ask for in the current conversation. Use apps only when the user requests a persistent tool.

For card payloads, app build rules, expanded card specs, and full platform documentation, read:
`reference/openpot-platform-guide.md`

<!-- END OPENPOT INSERT: platform-v2 -->
