<!-- OPENPOT INSERT: voice v1 — installed by openpot-awareness skill -->
<!-- DO NOT EDIT — managed by the OpenPot Awareness Skill -->

## OpenPot Voice Integration

OpenPot supports full voice input and output — mic-to-speech and
agent-to-voice — entirely on the client side. No server changes are
needed.

---

### Architecture

```
[Mic button] → Apple Speech (on-device STT) → text → WebSocket
→ Agent processes as normal text message
→ Agent text response → ElevenLabs TTS (client REST call)
→ MP3 playback on device
```

The agent receives and sends plain text. It has no awareness that the
user spoke — it processes voice input identically to typed input. You
do not need to change any server code, endpoints, or SOUL.md to support
voice. The feature lives entirely in the app.

---

### Voice Input

The mic button appears in the Chat input bar and in the AgentChatStrip
(the chat overlay on Pulse, Calendar, Apps, and Terminal tabs).

Tap and hold the mic button to record. OpenPot uses Apple's on-device
`SFSpeechRecognizer` — no audio leaves the device during transcription.
The transcribed text appears in the input field and is sent when released.

Requires iOS speech recognition permission (prompted on first use).

---

### Voice Output Modes

The user configures voice output in OpenPot **Settings → Voice Output**:

| Mode | Behavior |
|------|---------|
| **ElevenLabs** | High-quality TTS via ElevenLabs API. Requires API key. |
| **System Voice** | Apple `AVSpeechSynthesizer` — free, works offline, lower quality. |
| **Off** | No voice output. Text-only responses. |

---

### ElevenLabs Setup (User Configures in OpenPot)

ElevenLabs credentials are entered by the user in the OpenPot app —
not on the server. Each device stores its own credentials.

**In OpenPot Settings:**

1. Go to **Settings → Voice Output**
2. Select **ElevenLabs**
3. Enter your **ElevenLabs API Key** (from elevenlabs.io → Profile → API Key)
4. Enter a **Voice ID** (from your ElevenLabs voice library)
5. Select a **Model** (default: `eleven_turbo_v2_5` for low latency)

The API key is stored in the iOS Keychain — it is never sent to the
agent's server.

**Finding a Voice ID:**
- Go to elevenlabs.io → Voices
- Click any voice → click the ID icon to copy the Voice ID
- Use `eleven_turbo_v2_5` for low latency or `eleven_multilingual_v2`
  for multilingual support

---

### Per-Agent Voice Settings

Each agent can have its own Voice ID and playback speed. This is useful
if you have multiple agents with different personalities.

**In OpenPot Agents tab:**
1. Tap your agent
2. Scroll to **Voice** section
3. Enter a **Voice ID** (overrides the global default for this agent)
4. Set **Playback Speed** (0.5x – 2.0x, default 1.0x)

If no per-agent Voice ID is set, the global default from Settings is used.
If no global Voice ID is set, ElevenLabs TTS is disabled for that agent
(falls back to System Voice if enabled, otherwise silent).

---

### Zero Server Changes Required

Voice is entirely client-side. The flow is:

1. OpenPot calls ElevenLabs REST API directly from the device
2. ElevenLabs returns an MP3 audio stream
3. OpenPot plays the audio via `AVAudioPlayer`

Your agent server is not involved in TTS. You do not need to:
- Install anything on the server
- Add any endpoints
- Configure any environment variables for voice
- Modify SOUL.md for voice behavior

The only server-adjacent thing to be aware of is the Talk Mode warning below.

---

### Talk Mode Warning

OpenClaw has a Talk Mode protocol (`operator.talk.secrets` scope) that
routes TTS through the gateway. **Do NOT enable Talk Mode** on your
OpenPot setup. It interferes with `.final_` message delivery and causes
responses to arrive as incomplete fragments.

OpenPot uses direct client-side ElevenLabs calls, which work correctly
and do not require Talk Mode.

If you see this in your gateway config:
```json
"scopes": ["operator.talk.secrets"]
```
Remove it. Then restart the gateway.

---

### What the Agent Should Know About Voice

The agent receives voice input as plain text — identical to a typed
message. You do not need to adjust your SOUL.md or instructions for
voice behavior unless you want to change response length or style for
voice interactions.

If users ask you to read something aloud: they mean OpenPot's voice
output will speak your text response. Keep your answer as clear text —
OpenPot handles the TTS. Do not add SSML, markup, or special formatting
for voice.

---

### Troubleshooting

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| Mic button doesn't respond | iOS speech permission denied | Settings → Privacy → Speech Recognition → enable OpenPot |
| Mic records but sends nothing | Background noise below threshold | Speak more clearly; check mic is not blocked |
| ElevenLabs plays nothing after agent response | API key not set or wrong | Settings → Voice Output → re-enter API key |
| Wrong voice plays | Per-agent Voice ID not set, falling back to global | Agents tab → tap agent → set Voice ID |
| Audio too fast or too slow | Playback speed misconfigured | Agents tab → tap agent → adjust Playback Speed |
| System Voice broken on iPadOS beta | Known AVSpeechSynthesizer regression in some betas | Switch to ElevenLabs or wait for iPadOS update |
| 30-second delay before voice plays | Agent response taking too long | Check gateway latency; force-finalized streams still play after 30s |
| "Invalid API key" from ElevenLabs | Key expired or incorrect | Regenerate key at elevenlabs.io → Profile → API Key |
| Audio cuts off mid-sentence | Stream ended before full audio | Known issue with very long responses; agent should keep responses concise |

<!-- END OPENPOT INSERT: voice v1 -->
