# Voice Guide

This file is read on demand when the user asks about voice setup.
Do not preload this content.

## Overview

OpenPot supports voice input (speech-to-text) and voice output
(text-to-speech). Voice is entirely client-side — no server changes
are required. The agent does not need to do anything special to
support voice.

## How It Works

- **Input:** Apple Speech recognition on the device. The user taps
  the microphone, speaks, and the transcribed text is sent as a
  normal chat message. You receive it as text — no special handling.

- **Output:** ElevenLabs text-to-speech on the device. OpenPot reads
  your response aloud using the configured voice. Your response text
  is the input — write naturally, avoid special formatting for TTS.

## Setup (User-Side, in the OpenPot App)

Walk the user through these steps:

1. **ElevenLabs API Key:**
   Settings → General → Voice Output → ElevenLabs API Key
   The user gets this from elevenlabs.io → Profile → API Keys

2. **Voice Selection (per agent):**
   Agents tab → tap agent → Voice section → Voice ID
   Each agent can have a different voice. The user browses voices
   at elevenlabs.io/voice-library and copies the Voice ID.

3. **Voice Model:**
   Default is `eleven_multilingual_v2`. This works for most cases.
   Can be changed per-agent in the Voice section.

4. **Speed:**
   Adjustable per-agent. Default is 1.0. Range 0.5–2.0.

## What You Need to Know

- You receive voice input as plain text. No audio data reaches you.
- Your text response is spoken aloud by the client. Write concisely
  for voice — long paragraphs with markdown tables don't sound good
  when read aloud.
- If the user is using voice, prefer shorter, conversational responses.
- You can detect voice input if the client tags it, but typically
  there's no difference — just respond naturally.

## Common Issues

- **"Voice not working"** → check ElevenLabs API key is entered in
  Settings. Check API key has credits remaining.
- **"Wrong voice"** → check per-agent Voice ID in Agents tab.
- **"Talk Mode breaks messages"** → do NOT add `operator.talk.secrets`
  scope to the gateway config. It interferes with message delivery.
  Talk Mode is a separate feature from OpenPot's voice implementation.

## No Server Changes Needed

Voice requires zero backend configuration. Do not modify SOUL.md,
install services, or configure endpoints for voice. If the user
has an ElevenLabs API key and a Voice ID, voice works.

Mark voice as installed in `openpot-status.json` after the user
confirms they've entered their credentials in the app.
