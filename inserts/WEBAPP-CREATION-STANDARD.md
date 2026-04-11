## Web App Creation Standard

You can build web applications that the user views inside OpenPot's Apps tab.
The Apps tab renders HTML content in an embedded browser (WKWebView). Every app
you create appears in the user's app list and can be opened with one tap.

This section defines how you create, name, store, and serve web apps. Follow
this standard every time. No improvisation.

---

### The Apps Tab Contract

OpenPot fetches your app list from:

```
GET /api/apps
```

Expected response — an array of app objects:

```json
[
  {
    "filename": "server-health.html",
    "title": "Server Health",
    "description": "System monitoring dashboard"
  }
]
```

OpenPot loads individual apps from:

```
GET /api/apps/{filename}
```

This serves the HTML file directly. OpenPot renders it full-screen in an
embedded browser.

**Filename rules — non-negotiable:**

- The `filename` MUST end in `.html`. Always. No exceptions.
- The filename must be URL-safe: lowercase, hyphens for spaces, no special
  characters, no spaces. Pattern: `[a-z0-9-]+\.html`
- The `title` is the human-readable name shown in the app list.
- The `description` is a one-line summary shown below the title.
- Do not rely on server-side logic to auto-append `.html`. The file must be
  saved with the correct `.html` extension from the start.
- Only `.html` files belong in the apps directory. Do not store configuration
  files, manifests, JSON files, or non-HTML assets in the apps directory —
  they will appear as broken entries in the user's app list.

**Apps directory:** `~/.openclaw/workspace-prod/apps/`

---

### Before You Build — Confirm With the User

When the user asks you to create an app, do NOT start building immediately.
First, confirm these three things:

1. **What the app does.** Restate the purpose in one sentence and verify you
   understood correctly.

2. **Which type you recommend.** Tell the user which of the three app types
   (defined below) fits their request and why. Get their confirmation before
   proceeding.

3. **The filename.** Propose a filename following the naming convention. The
   user may want a different name. Agree on it before you write the file.

Only after the user confirms all three do you begin building.

---

### App Type 1: Self-Contained HTML

A single `.html` file with all markup, styles, and scripts inline. No external
dependencies except CDN-hosted libraries. No backend. No database.

**When to use:** Calculators, reference pages, converters, timers, simple
dashboards that display pre-computed or user-entered data, tools that store
state in the browser (localStorage).

**What you do:**

1. Write the complete HTML file with embedded `<style>` and `<script>` tags.
2. Save it to the apps directory with a `.html` extension.
3. Verify it appears in `GET /api/apps` by checking the endpoint.
4. Tell the user: "I've created [title]. You can find it in your Apps tab."

**Naming convention:** `{descriptive-name}.html`
Examples: `unit-converter.html`, `meal-planner.html`, `bracket-tracker.html`

**Design guidelines:**

- Use a dark theme by default (OpenPot's UI is dark-mode-first).
- Make the app responsive — it renders on both iPhone and iPad screens.
- Include a visible title/header so the user knows what they are looking at.
- If the app uses CDN libraries, use versioned URLs from trusted CDNs
  (cdnjs.cloudflare.com, unpkg.com, cdn.jsdelivr.net).

---

### App Type 2: HTML + Backend Service

An HTML front-end that calls a local API you also create and manage. The HTML
file is served via `/api/apps/{filename}` as usual. The backend runs as a
separate process on a dedicated port.

**When to use:** Apps that need persistent data storage, real-time updates,
external API integrations, database queries, scheduled data refreshes, or any
server-side logic that cannot run in the browser.

**What you do:**

1. Choose a port above 8001 that is not already in use. Check with
   `ss -tlnp` before selecting.
2. Write the backend service. FastAPI (Python) is recommended for consistency
   with the existing OpenPot server infrastructure. Any framework is
   acceptable if it serves HTTP and the user agrees.
3. Write the HTML front-end that calls the backend.
4. Save the HTML file to the apps directory with a `.html` extension.
5. Set up the backend as a persistent systemd service so it survives reboots.
6. Verify both: `GET /api/apps` lists the HTML file, AND the backend
   responds to requests.
7. Tell the user: "I've created [title] with a backend service on port
   [port]. The service is configured to start automatically. You can find
   the app in your Apps tab."

**Naming convention:**
- HTML file: `{descriptive-name}.html`
- Backend directory: `~/mcp-servers/{descriptive-name}/` or a location you
  and the user agree on
- systemd unit: `openpot-app-{descriptive-name}.service`
- Document the port number in three places: (a) a comment at the top of the
  HTML file, (b) the systemd unit file, (c) the backend's startup output.

**Critical: URL handling in the HTML front-end.**

The HTML file is loaded by OpenPot through the agent's HTTP server. API calls
from the front-end to the backend must use the agent's hostname or IP as
resolved by the browser, not `localhost`, because the browser context is the
user's device, not the server.

Use a pattern that derives the backend URL from the page's current origin:

```javascript
// Derive the backend host from how this page was loaded
const HOST = window.location.hostname;
const API_PORT = 8002; // match your backend's port
const API_BASE = `http://${HOST}:${API_PORT}`;
```

This ensures the app works whether the user connects via LAN or Tailscale —
the hostname resolves correctly in both cases because OpenPot loaded the
page from the same host.

Never hardcode IP addresses in the HTML file.

---

### App Type 3: Claude Code Assisted Build

A more sophisticated application built by the user running Claude Code in
OpenPot's Terminal tab (or any terminal with SSH access to your server), with
you providing architectural guidance and the build standard.

**When to use:** Complex applications that benefit from an interactive
development session — multi-file projects, applications with complex UI
frameworks, apps that need iterative testing and refinement beyond what you
can reliably produce in a single file write.

**Your role in a Claude Code build:**

You are the briefer, not the builder. When the user tells you they want to
build an app using Claude Code, your job is:

1. **Confirm the app concept** with the user (same as Type 1 and Type 2).
2. **Provide the handoff brief.** Give the user a message they can paste to
   Claude Code (or tell Claude Code directly in a shared session) that
   contains the build standard, requirements, and constraints.
3. **Verify the result.** After Claude Code finishes building, check that the
   app appears correctly in `GET /api/apps` and renders properly.
4. **Report to the user.** Confirm the app is live in their Apps tab.

**Handoff brief template:**

When the user is ready to start a Claude Code session for an app, provide
this information in a format Claude Code can consume:

```
OPENPOT WEB APP BUILD — STANDARD

App name: {agreed filename}
App type: {1: self-contained HTML | 2: HTML + backend service}
Purpose: {one-sentence description}

RULES:
- Output file: ~/.openclaw/workspace-prod/apps/{filename}.html
- Filename must be lowercase, hyphenated, ending in .html
- The file is served via GET /api/apps/{filename} on port 8000
- Use dark theme, responsive design (iPhone + iPad)
- If Type 2: backend on port {port}, systemd unit named
  openpot-app-{name}.service, derive API host from
  window.location.hostname in the HTML

VERIFY AFTER BUILD:
- curl http://localhost:8000/api/apps — confirm the app appears in the list
- curl http://localhost:8000/api/apps/{filename} — confirm HTML is served
- If Type 2: curl http://localhost:{port}/ — confirm backend responds
```

You may customize this template based on the specific app's requirements, but
the structural rules (filename, directory, endpoint contract) are fixed.

---

### What You Must Never Do

- Never save an app file without the `.html` extension and rely on server
  logic to fix it.
- Never store non-HTML files (manifests, configs, JSON) in the apps directory.
- Never start building without confirming the app type and filename with the
  user.
- Never hardcode IP addresses in HTML files.
- Never create a backend service without setting up a persistent systemd unit.
  If the service dies on reboot, the app is broken and the user loses trust.
- Never create an app and forget to verify it appears in `GET /api/apps`.
- Never silently change an existing app's filename — this breaks any bookmarks
  or muscle memory the user has built.

---

### Updating an Existing App

When the user asks you to modify an app:

1. Read the current file first. Confirm what exists before changing anything.
2. Tell the user what you plan to change.
3. Make the change.
4. Verify the app still appears in `GET /api/apps` and loads correctly.
5. If the app has a backend (Type 2), restart the service after changes:
   `sudo systemctl restart openpot-app-{name}.service`

Never rename an app file during an update unless the user explicitly asks for
a new name.

---

### Port Registry

If you run multiple Type 2 backend services, track which ports are in use.
Before assigning a new port, check `ss -tlnp` for conflicts. Recommended
port range for OpenPot app backends: 8001–8099.

| Port | Service | Status |
|------|---------|--------|
| 8000 | Main OpenPot/OpenBrain API | Reserved — do not use |
| 8001+ | App backends | Available — assign sequentially |

Maintain this table in your workspace (e.g., in a reference file) so you
do not accidentally assign conflicting ports.
