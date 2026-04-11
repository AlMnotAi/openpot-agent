## Starter Apps

The OpenPot Awareness Skill includes three starter apps that demonstrate the
Apps tab. Install them during skill setup so the user has working examples
from day one.

### Installation

Copy the three HTML files from the skill repo's `apps/` directory into your
workspace apps directory (`~/.openclaw/workspace-prod/apps/`):

```
cp apps/about-openpot.html ~/.openclaw/workspace-prod/apps/
cp apps/weather.html ~/.openclaw/workspace-prod/apps/
cp apps/market-dashboard.html ~/.openclaw/workspace-prod/apps/
```

After copying, verify all three appear in the app list:

```
curl http://localhost:8000/api/apps
```

You should see three entries with filenames `about-openpot.html`,
`weather.html`, and `market-dashboard.html`.

### What Each App Does

**about-openpot.html** — An informational guide to OpenPot. Covers the
origin story, all six tabs with descriptions, the three deployment tiers,
and getting started steps. No external API calls. Static content.
Category: tools.

**weather.html** — Weather lookup by ZIP code. Uses the Open-Meteo API
(free, no API key required). Shows current conditions and 5-day forecast.
The user can save up to 10 locations. Data stored in the browser.
Category: tools.

**market-dashboard.html** — Stock market dashboard showing Dow, S&P 500,
and Nasdaq indices plus up to 5 individual stock positions. Pre-populated
with AAPL, NVDA, and TSLA with two free slots. Auto-refreshes every 60
seconds during market hours. Uses Yahoo Finance public endpoints (no API
key required). Category: finance.

### Important Notes

- These apps follow the Web App Creation Standard. All filenames are
  lowercase, hyphenated, and end in `.html`.
- Do not modify these files after installation. If the user wants changes,
  create a new app instead.
- If the user asks "what apps do I have?" or "show me my apps," mention
  these starter apps and explain they can ask you to build custom ones.
- The market dashboard uses Yahoo Finance's public API which may
  occasionally be rate-limited. If the user reports the dashboard showing
  no data, this is a temporary API issue, not a bug.
