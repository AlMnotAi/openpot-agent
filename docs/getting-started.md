# Getting Started with OpenPot

You've got OpenPot on your phone and your agent is connected. Now what?

These are things you can say to your agent — in OpenPot chat, Telegram, Slack, wherever you talk to it. Your agent does the work.

---

## First Things First

**Set up OpenPot features:**

> "Install the OpenPot Awareness Skill from https://github.com/AlMnotAi/openpot-agent"

> "OpenPot sync"

Your agent installs the skill, checks what it can support, and tells you what's available. When new features are added to OpenPot, just say "OpenPot sync" again and your agent picks them up.

---

## Explore the Pulse Tab

The Pulse tab shows cards your agent sends you — proactive updates, reminders, reports, alerts. It's empty until your agent starts posting cards.

**See what it looks like:**

> "Send me a few sample Pulse cards so I can see how they work"

> "Send me a test card for each category — finance, calendar, health, system, email, projects, entertainment, reminder, and briefing"

**Set up a daily briefing:**

> "Every morning at 7 AM, send me a Pulse card with today's weather, my calendar, and anything I need to know"

**Set up reminders:**

> "Remind me every Monday at 9 AM to review my budget"

> "Send me a card if any of my stocks drop more than 5% in a day"

**Proactive monitoring:**

> "Check the server health every hour. If anything is wrong, send me a system card"

> "Watch my email. If something looks urgent, send me a card about it"

### Tap to Expand

Report-type cards (system health, DCA signals, morning briefs) are tappable. Look for the small chevron on the card — tap it to open a full detail sheet with the complete report. The card face is the summary, the detail sheet is the full story.

If your cards don't expand, tell your agent:

> "OpenPot sync"

The sync will install the expansion feature directives so your agent starts including full reports in its cards.

---

## Explore the Calendar Tab

The Calendar tab shows events your agent tracks — meetings, deadlines, cron job schedules, reminders, and anything else time-bound. Your agent is the source of truth; the calendar displays what it knows.

**Populate your calendar:**

> "Add my usual Monday portfolio review to my OpenPot calendar — every Monday at 4:30 PM"

> "Track my upcoming appointments in the OpenPot calendar"

> "Add a reminder for my oil change — it's due April 20"

**Switch between views:**

The Calendar tab has two views you can toggle between:

- **Month** — grid view with dot indicators on days that have events
- **Agenda** — chronological list from today through the end of next month, grouped by day with sticky date headers

If your Calendar tab is empty, your agent needs the calendar API endpoint set up. Tell your agent:

> "OpenPot sync"

---

## Try the Apps Tab

Your agent can build small web apps that live in the Apps tab. These are tools, dashboards, and reference pages — built on demand.

> "Build me a unit converter app — weight, distance, and temperature"

> "Build me a tip calculator app"

> "Build me a medication schedule tracker"

> "Build me a simple dashboard that shows my server's uptime and status"

> "Build me a dark-themed grocery list app"

The app shows up in the Apps tab. Tap to open it full-screen.

---

## Chat Tips

OpenPot's chat works like any conversation with your agent, but it's on your phone — always with you.

**Quick tasks:**

> "What's on my calendar today?"

> "Summarize my unread emails"

> "What's the weather this weekend?"

**Have your agent remember things:**

> "Remember that my anniversary is June 14"

> "Remember that I take metformin 500mg twice daily"

> "Remember that Mark's favorite restaurant is Leo's on Main Street"

**Ask about what it knows:**

> "What do you know about my medication schedule?"

> "When is my anniversary?"

---

## Skills

The Agents tab → tap your agent → Skills shows what capabilities your agent has — web search, email, calendar access, file tools, and more. You don't need to configure anything here, but it's useful to see what's available.

> "What skills do you have?"

> "Can you search the web?"

> "Can you read my email?"

---

## Terminal Tab

If you're technical, the Terminal tab gives you a direct SSH connection to your server — right from your phone. Useful for quick checks without opening a laptop.

---

## Settings

Tap the **gear icon** at the bottom of the agent dock (the left sidebar) to open Settings. This is where you manage gateway connections, connection routes, and app appearance.

---

## Agent Themes

Each agent can have its own color theme so you can tell at a glance which agent you're talking to. Go to the Agents tab, tap your agent, and pick a theme from the color swatches. The theme tints the chat bubbles, sidebar, and dock indicator.

---

## Multiple Agents

OpenPot supports multiple agents. Each agent gets its own dock icon on the left side of the screen. Tap an agent to switch — the entire app context changes to that agent's chat, cards, apps, and calendar.

To add a second agent, go to the Agents tab and tap the + button. You'll need the gateway address and token for the new agent (see the connection guide).

---

## Make It Yours

Tell your agent what you care about and it'll start sending you the right cards, building the right tools, and keeping track of the right things.

> "I care about fitness, cooking, and my investment portfolio. Keep those in mind when you send me cards."

> "I work night shifts Tuesday through Friday. Don't send me morning briefings on those days."

> "I'm training for a half marathon. Track my runs when I tell you about them."

Your agent learns from what you tell it. The more context you give, the more useful OpenPot becomes.
