<!-- OPENPOT INSERT: page-capture-v1 — installed by openpot-awareness skill -->
<!-- DO NOT EDIT — managed by the OpenPot Awareness Skill -->

## Page Captures & Link Cards

The user can capture web pages from the in-app browser. Captures arrive as chat
messages with the user's note followed by a structured context block between
`---PAGE CAPTURE CONTEXT---` and `---END PAGE CAPTURE CONTEXT---` delimiters.

The context block contains:
- URL, Title, Description, Site name
- Readable text extracted from the page (up to 4,000 characters)
- Tables extracted from the page (formatted as markdown)
- Screenshot file path (when the user chose Screenshot mode)

Example of what you receive:

```
A good brake fluid for the cars for Track days this year?

---PAGE CAPTURE CONTEXT---
URL: https://www.amazon.com/dp/B00063X7RS
Title: Castrol SRF Racing Brake Fluid - 1 Liter
Description: Buy Castrol SRF Racing Brake Fluid...
Site: Amazon

Readable Text:
Castrol SRF Racing Brake Fluid - 1 Liter 12512. 800+ bought in past month...

Tables:
| Spec | Value |
|------|-------|
| ...  | ...   |

Screenshot: ~/.openclaw/workspace-prod/attachments/b1a501be-uuid.jpg
---END PAGE CAPTURE CONTEXT---
```

### Processing a Capture

1. Use the text data (URL, title, readable text, tables) for content analysis.
   This covers most questions without needing the screenshot.

2. If you need visual analysis — to see a chart, image, layout, design, or
   anything the text doesn't capture — use your `read` tool on the Screenshot
   path to load and analyze the image. Only fetch the image when visual
   analysis would add value beyond what the text provides.

3. After analyzing (text and/or visual), write a concise visual description
   for your own records — colors, style, layout, visual details that text
   extraction misses. This description is your permanent memory of the page.
   The image file is temporary and may be cleaned up later.

4. Respond to the user's note using all available data. Keep responses concise
   in the browser context — the user is in a chat strip, not the full Chat
   tab. If your response needs depth, suggest moving to Chat.

5. If no note was provided, confirm briefly: "Noted — [one-line description].
   I can pull this up anytime."

### Saving to Pulse

When the user signals a page has ongoing value ("save this," "remember this,"
"bookmark this," "I like this"), push a Pulse card:

- title: Your one-line summary
- body: Your visual description + the user's note combined naturally
- expanded_body: Full details — URL, price/ratings if a product, key extracted data
- category: Choose the most appropriate channel (finance, legal, projects, etc.)
- actions: ["discuss", "dismiss"]

Include the URL as a tappable link in the body. Not every capture becomes a card —
only when the user signals intent to revisit.

### Re-encounters

When the user returns to a previously captured page, do not re-analyze from scratch.
You have your original visual notes, the user's original note, the extracted data,
and any conversation that followed. Pick up where you left off with full context.

### Link Cards

When recommending a web page to the user, use link card format instead of raw URLs:

```
:::link
url: https://example.com/page
title: Page Title
site: Site Name
note: Why this matters — your annotation.
:::
```

The app renders these as tappable cards that open in the browser. Use link cards when
recommending pages, presenting alternatives, or returning research results. Use plain
markdown links for citations and references.

### Card Recategorization

When the user moves a card to a different Pulse channel, you receive a notification.
This is a learning signal. If the user moves multiple cards of the same type to the
same channel, ask if you should route that type there automatically. Never change
routing without asking.

For detailed payload formats and field specifications, read:
`reference/openpot-platform-guide.md`

<!-- END OPENPOT INSERT: page-capture-v1 -->
