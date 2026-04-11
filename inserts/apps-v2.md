## OpenPot Apps Tab

OpenPot has an **Apps tab** where the user accesses web applications you
create. Apps open full-screen inside an embedded browser. The tab uses a
bottom drawer for navigation and displays apps in a categorized grid.

### How the User Sees It

When the user opens the Apps tab:

- If no apps exist: they see an onboarding screen explaining how to get
  apps built (by asking you or using Claude Code in the Terminal tab).
- If apps exist: they see a bottom drawer (peek state) showing their most
  recent apps. They can drag the drawer up to see all apps organized by
  category in a grid. Tapping an app opens it full-screen with a top bar
  (back button, title, overflow menu with Reload, Open in Safari, About,
  and Report Issue).

### Apps API Contract

**List apps:**
```
GET /api/apps
```

Returns an array of app objects:
```json
[
  {
    "filename": "weather.html",
    "title": "Weather",
    "description": "Current conditions and 5-day forecast",
    "category": "tools"
  }
]
```

**Serve an app:**
```
GET /api/apps/{filename}
```

Returns the raw HTML file for rendering in the embedded browser.

### Categories

Every app has a `category` that determines its grouping and color in the
grid. Use these canonical categories:

- **tools** — calculators, converters, reference, utilities
- **health** — medication tracking, fitness, wellness
- **finance** — portfolio, DCA, budgeting, market data
- **projects** — project dashboards, task trackers, milestone views
- **monitoring** — server health, service status, uptime
- **entertainment** — media, sports, leisure, games
- **uncategorized** — default when no category fits

When creating an app, always assign a category. If the user's request
does not fit an existing category, propose a new one and use it
consistently for similar apps in the future.

### App Types

There are three types of apps you can create. Know which one fits before
you start building, and confirm with the user.

**Type 1: Self-Contained HTML**
A single `.html` file with all markup, styles, and scripts inline. No
backend, no database. Use for calculators, reference pages, converters,
simple dashboards. Save directly to the apps directory.

**Type 2: HTML + Backend Service**
An HTML front-end that calls a local API you also create and run. The
backend runs as a separate process on a dedicated port (8001+). Set it
up as a persistent systemd service. Use for apps needing persistent
data, real-time updates, API integrations, or database access.

**Type 3: Claude Code Assisted Build**
A more sophisticated app built by the user running Claude Code in the
Terminal tab. You provide the architectural brief and the build standard.
Claude Code does the building. You verify the result.

### Before You Build — Confirm With the User

When the user asks for an app, confirm before starting:

1. **What it does** — restate the purpose in one sentence.
2. **Which type** — tell the user which type fits and why.
3. **The filename** — propose a URL-safe, hyphenated `.html` filename.
4. **The category** — propose a category from the canonical list.

Only begin building after the user confirms all four.

### File Rules

- Filename MUST end in `.html`. Lowercase, hyphens only.
  Pattern: `[a-z0-9-]+\.html`
- Only `.html` files in the apps directory. No manifests, configs, or
  backup files.
- Apps directory: `~/.openclaw/workspace-prod/apps/`
- Do not rely on server-side logic to fix filenames. Save correctly
  from the start.

### Design Guidelines

- Dark theme by default (OpenPot is dark-mode-first)
- Responsive — works on iPhone (375pt) and iPad screens
- Include a visible title/header
- CDN libraries are fine — use versioned URLs from trusted CDNs

### Type 2 Backend Rules

- Choose a port above 8001. Check `ss -tlnp` first.
- Set up as a persistent systemd service.
- Derive the API host from `window.location.hostname` in the HTML —
  never hardcode IP addresses.
- Document the port in the HTML file header, systemd unit, and backend
  startup output.
- Service name: `openpot-app-{name}.service`

### Updating an Existing App

1. Read the current file first.
2. Tell the user what you plan to change.
3. Make the change.
4. Verify the app still appears in `GET /api/apps`.
5. If Type 2, restart the backend service.
6. Never rename a file during an update unless the user asks.

### Starter Apps

Three apps ship with the OpenPot Awareness Skill and should be installed
during setup: `about-openpot.html` (informational guide), `weather.html`
(Open-Meteo weather lookup), and `market-dashboard.html` (stock market
indices and positions). Copy them from the skill repo's `apps/` directory
to the workspace apps directory during installation.

### What You Must Never Do

- Never save without `.html` extension
- Never store non-HTML files in the apps directory
- Never build without confirming type, filename, and category
- Never hardcode IP addresses in HTML files
- Never create a Type 2 backend without a persistent systemd service
- Never skip verification via `GET /api/apps` after creating
- Never silently rename an existing app file
