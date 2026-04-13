# Security & Transparency

This document explains exactly what the OpenPot Awareness Skill does to your agent, why it does it, and how to verify, audit, and reverse every change.

---

## What This Skill Modifies

When you install this skill, three types of changes are made to your agent's workspace:

| What | Where | Why |
|------|-------|-----|
| Behavioral inserts | `workspace/SOUL.md` | Teaches your agent how to interact with OpenPot's iOS surfaces — how to format Pulse cards, handle page captures, use link cards, and respond in the browser chat strip |
| Reference files | `workspace/reference/` | Detailed specifications your agent reads on demand — card payload formats, app build rules, design guidelines. Not injected into SOUL.md. |
| Starter apps | Served via `/api/apps` | HTML files your agent serves to OpenPot — About OpenPot, weather, market dashboard. Self-contained, no external network calls. |

---

## Why SOUL.md?

This is the question that matters. SOUL.md is your agent's identity and behavioral core. Modifying it is not something we do lightly, and you should understand why a reference file alone is not sufficient.

**SOUL.md is the agent's bootstrap prompt.** It loads every time the agent starts a conversation. Instructions here are always active — the agent doesn't need to be told to read them, they're already in context.

**Reference files are on-demand.** The agent reads them when it needs detailed specs (card payload formats, app build rules). But the agent needs to know *that it should* read them, and *when* to do things like format link cards or push Pulse cards. Those behavioral triggers must be in the bootstrap prompt.

**What cannot live in a reference file:**
- "When you receive a page capture, write visual notes" — the agent needs this in its active context to act on captures as they arrive
- "When the user says save this, push a Pulse card" — this is a behavioral directive, not a spec to look up
- "Use `:::link` format when recommending pages" — the agent needs to know this during response generation, not after

**What lives in reference files instead:**
- Card POST payload field specifications
- App build rules and design guidelines
- Detailed format examples and edge cases

The split is intentional: behavioral directives in SOUL.md (~500-1,800 chars per insert), detailed specs in reference files (unlimited size, read on demand).

---

## What the Inserts Do NOT Do

The inserts in this skill:

- **Do not change your agent's identity.** Your agent's name, personality, tone, and character are untouched. The inserts add capabilities, not identity.
- **Do not override safety rules.** Your agent's existing safety boundaries, content policies, and behavioral guardrails are unaffected.
- **Do not modify the memory pipeline.** Your agent's memory capture, storage, and retrieval instructions are untouched. (Earlier versions of this skill had a bug where large inserts pushed memory instructions below the truncation line — this was fixed in v5.0 with the compact stub architecture.)
- **Do not route data to external services.** No insert instructs your agent to send data anywhere outside your own infrastructure. All communication stays between your agent, your gateway, and your OpenPot app on your phone.
- **Do not install cron jobs, background tasks, or persistent processes.** The skill only writes files. Your agent's operational behavior is unchanged outside of conversations.
- **Do not access, read, or modify any files outside the designated paths** (SOUL.md, reference/, and the apps directory).

---

## How to Audit What Was Changed

### View the SOUL.md inserts

Every insert is wrapped in clearly marked boundaries:

```
<!-- OPENPOT INSERT: [insert-name] — installed by openpot-awareness skill -->
<!-- DO NOT EDIT — managed by the OpenPot Awareness Skill -->

[insert content here]

<!-- END OPENPOT INSERT: [insert-name] -->
```

To see all OpenPot inserts in your agent's SOUL.md:

```bash
grep -A 999 "OPENPOT INSERT" workspace/SOUL.md | grep -B 999 "END OPENPOT INSERT"
```

Or simply open `workspace/SOUL.md` in any text editor and search for "OPENPOT INSERT."

### View reference files

```bash
ls workspace/reference/
cat workspace/reference/openpot-platform-guide.md
```

### View starter apps

```bash
ls workspace/apps/  # or wherever your agent serves app files
```

### Compare against the source

Every file installed by this skill exists in this public GitHub repository. You can diff your installed files against the repo to verify nothing was modified during installation:

```bash
diff workspace/reference/openpot-platform-guide.md <(curl -s https://raw.githubusercontent.com/AlMnotAi/openpot-agent/main/reference/openpot-platform-guide.md)
```

---

## How to Remove the Skill

### Full uninstall

If your agent supports skill uninstall:

```
"Uninstall the OpenPot Awareness Skill"
```

This removes the SOUL.md inserts (everything between the markers), the reference files, and the starter apps.

### Manual removal

1. Open `workspace/SOUL.md`
2. Find each `<!-- OPENPOT INSERT: ... -->` block
3. Delete everything from the opening marker through the closing `<!-- END OPENPOT INSERT: ... -->` marker
4. Delete `workspace/reference/openpot-platform-guide.md`
5. Optionally delete starter app files

Your agent reverts to its pre-install behavior on the next conversation.

### Selective removal

You can remove individual inserts without uninstalling the entire skill. Delete only the specific `<!-- OPENPOT INSERT: [name] -->` block you want to remove. The remaining inserts continue to function independently.

---

## Current Inserts (as of v6.0)

| Insert | File | Chars | Purpose |
|--------|------|-------|---------|
| `platform-v2` | `inserts/platform-v2.md` | ~500 | Core awareness — tells the agent OpenPot exists, names the three output surfaces (Chat, Pulse, Apps), points to the reference file |
| `calendar-v2` | `inserts/calendar-v2.md` | ~800 | Calendar integration — how to query and format calendar events for OpenPot's Calendar tab |
| `voice-v1` | `inserts/voice-v1.md` | ~400 | Voice interaction — how to handle voice input and format responses for TTS |
| `page-capture-v1` | `inserts/page-capture-v1.md` | ~1,800 | Page capture — how to process browser captures, write visual notes, use link cards, save to Pulse, learn from recategorization |

**Total SOUL.md impact:** ~3,500 characters across all inserts. Your agent's `bootstrapMaxChars` limit is 20,000. The inserts use approximately 18% of the available space.

---

## Trust Model

This skill operates on a trust-but-verify model:

1. **Every file is public.** The complete source is on GitHub. Nothing is hidden, encrypted, or obfuscated.
2. **Every change is marked.** SOUL.md modifications are wrapped in markers that identify the skill and insert version.
3. **Every change is reversible.** Uninstall removes everything cleanly.
4. **Every change is auditable.** You can diff installed files against the public repo at any time.
5. **The skill does not phone home.** No analytics, no telemetry, no external network calls. Your agent talks to your gateway and your OpenPot app. That's it.

If you find a discrepancy between what this document describes and what the skill actually does, please open an issue on GitHub. Transparency is not optional — it's the foundation of the trust relationship between a skill and an agent owner.

---

## Questions or Concerns

If you have security questions about this skill:

- **Open a GitHub issue** on this repository
- **Review the source code** — every file is public and documented
- **Ask your agent** to read and summarize the inserts before installing: "Read the OpenPot Awareness Skill inserts from [repo URL] and tell me exactly what they would add to my SOUL.md"
