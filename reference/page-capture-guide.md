# Page Capture Guide

This file is read on demand when you receive a page capture from the
OpenPot in-app browser. Do not preload this content.

## What You Receive

The user can capture web pages from OpenPot's in-app browser. Captures
arrive as chat messages with the user's note followed by a structured
context block:

```
[User's note here]

---PAGE CAPTURE CONTEXT---
URL: https://www.amazon.com/dp/B00063X7RS
Title: Castrol SRF Racing Brake Fluid - 1 Liter
Description: Buy Castrol SRF Racing Brake Fluid...
Site: Amazon

Readable Text:
[Up to 4,000 characters of extracted page content]

Tables:
| Header | Header |
|--------|--------|
| Value  | Value  |

Screenshot: ~/.openclaw/workspace/attachments/{uuid}.jpg
---END PAGE CAPTURE CONTEXT---
```

Fields included:
- **URL, Title, Description, Site** — page metadata
- **Readable Text** — main content body, up to 4,000 characters
- **Tables** — extracted HTML tables as markdown (if present)
- **Screenshot** — file path to the captured viewport image (if the
  user chose Screenshot mode; omitted in Data Only mode)

## Processing a Capture

1. **Use the text data first.** The URL, title, readable text, and
   tables cover most questions without needing the screenshot.

2. **For visual analysis,** use your `read` tool on the Screenshot
   path. This loads and analyzes the image. Only do this when the
   question requires seeing the page — charts, images, layout,
   design, product photos, visual elements the text doesn't capture.

3. **Write a visual description** for your own records after analyzing.
   Example: "matte black single-handle kitchen faucet, pull-down
   sprayer, modern industrial style, brushed gunmetal finish."
   This description is your permanent memory of the page. The image
   file is temporary and may be cleaned up later.

4. **Respond to the user's note.** Keep responses concise — the user
   is in a chat strip inside the browser, not the full Chat tab.
   If your response needs depth (long comparison, detailed analysis),
   suggest moving to Chat.

5. **If no note was provided,** confirm briefly: "Noted — [one-line
   description]. I can pull this up anytime."

## Saving to Pulse

When the user signals a page has ongoing value — "save this,"
"remember this," "bookmark this," "I like this" — push a Pulse card:

- title: Your one-line summary
- body: Your visual description + the user's note, combined naturally
- expanded_body: Full details — URL, price/ratings if a product, key
  extracted data, your visual notes
- category: Most appropriate channel (finance, legal, projects, etc.)
- actions: ["discuss", "dismiss"]

Include the URL as a tappable link in the body. Not every capture
becomes a card — only when the user signals intent to revisit.

## Re-encounters

When the user returns to a previously captured page, do not re-analyze
from scratch. You have your original visual notes, the user's original
note, the extracted data, and any conversation that followed. Pick up
where you left off with full context.

## Link Cards

When recommending a web page to the user — a product at a different
vendor, a deeper link, an article from research — use this format:

```
:::link
url: https://example.com/page
title: Page Title
site: Site Name
note: Why this matters — your annotation.
:::
```

OpenPot renders these as tappable cards that open directly in the
in-app browser. Use link cards when:
- Recommending a specific page the user should look at
- Presenting alternatives (better price, different vendor)
- Returning research results

Use plain markdown links for citations and supplementary references.

## Card Recategorization

When the user moves a card to a different Pulse channel, you receive
a notification. This is a learning signal. If the user moves multiple
cards of the same type to the same channel, ask: "Want me to route
[type] to [channel] automatically?" Never change routing without asking.
