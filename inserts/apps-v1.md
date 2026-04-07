<!-- OPENPOT INSERT: apps v1 — installed by openpot-awareness skill -->
<!-- DO NOT EDIT — managed by the OpenPot Awareness Skill -->

## OpenPot Web Apps

OpenPot has an **Apps tab** that displays web apps you create as
full-width bars sorted by usage frequency. The user pulls down to
refresh the app list. Tapping an app opens it full-screen in WKWebView.

### How It Works

1. You create a self-contained HTML file
2. Save it to your workspace apps directory
3. Register it in the apps manifest
4. OpenPot fetches the list and displays it
5. User taps → app opens in a web view inside OpenPot

### Apps API

**List apps:**
```
GET /api/apps
```

Returns:
```json
[
  {
    "filename": "server-health.html",
    "title": "Server Health",
    "description": "System monitoring dashboard"
  }
]
```

**Serve an app:**
```
GET /api/apps/{filename}
```

Returns the raw HTML content.

### Creating Apps

When the user asks you to build a web app, create a single HTML file:

- **Self-contained** — all CSS and JS inline (CDN libraries are OK)
- **Dark theme** — match OpenPot's dark aesthetic
- **Mobile-first** — must work on iPhone widths (375–428pt)
- **Functional** — real utility, not placeholder content

Save to your workspace apps directory and ensure the manifest includes it.

<!-- END OPENPOT INSERT: apps v1 -->
