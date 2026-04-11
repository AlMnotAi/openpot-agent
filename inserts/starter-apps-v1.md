## Starter Apps

The OpenPot Awareness Skill includes six starter apps that ship with the
Apps tab. Install them during skill setup so the user has working examples
from day one.

### Installation

Copy all HTML files from the skill repo's `apps/` directory into your
workspace apps directory (`~/.openclaw/workspace-prod/apps/`):

```
cp apps/*.html ~/.openclaw/workspace-prod/apps/
```

After copying, verify all six appear in `GET /api/apps`.

### What Each App Does

**about-openpot.html** — An informational guide to OpenPot. Covers the
origin story, all six tabs with descriptions, what you need to get
started, and how to build custom apps. No external API calls. Static
content. Category: tools.

**weather.html** — Weather lookup by city name or US ZIP code. Uses
Zippopotam.us for US ZIP codes and Open-Meteo for city name search and
weather data (both free, no API key required). Current conditions and
5-day forecast. Stores up to 10 locations. Category: tools.

**focus-timer.html** — Pomodoro-style focus timer. Configurable focus
(25 min default), short break (5 min), and long break (15 min) intervals.
Tracks daily sessions, total focus minutes, and streak. Session history
stored locally. Category: tools.

**meeting-cost.html** — Real-time meeting expense calculator. Set number
of attendees and average hourly rate, start the timer, watch the cost
tick up live. Saves meeting history. Category: tools.

**subnet-calc.html** — IPv4 subnet calculator for IT professionals.
Enter an IP address and CIDR prefix, get network address, broadcast,
subnet mask, wildcard mask, usable host range, and binary
representation. Common subnet presets included. Category: tools.

**billable-hours.html** — Time tracker for consultants and freelancers.
Start/stop timer per client or project with hourly rate. Daily and
weekly summaries with earnings. Client list remembered across sessions.
Category: tools.

### Important Notes

- These apps follow the Web App Creation Standard. All filenames are
  lowercase, hyphenated, ending in `.html`.
- Do not modify these files after installation. If the user wants
  changes, create a new app instead.
- If the user asks "what apps do I have?" mention these starter apps
  and explain they can ask you to build custom ones.
- The weather app uses Zippopotam.us (US ZIP codes) and Open-Meteo
  (city search + weather data). Both are free with no API keys. If
  a lookup fails, suggest trying a city name instead of a postal code.
- All apps store user data in the browser's localStorage. Data persists
  across sessions but is specific to the device.
