# Meeting Copilot — Application Summary
**Version:** 0.1 (Draft)
**Date:** 2026-02-14

---

## What Is It?

Meeting Copilot is a **local desktop application** that listens to your meetings in real time — virtual or in-person — and acts as a silent strategic advisor. It transcribes the conversation, feeds it to an AI reasoning model along with client history and your product catalog, and surfaces smart questions, talking points, and contextual notes while you're still on the call.

After the meeting, it generates a full transcript, structured summary, action items, and suggested updates to your client records — all without sending a single thing to the cloud that you don't explicitly configure.

---

## Who Is It For?

Roy Bales and Jonathan Gough — technology consultants and Field CTOs at a value-added reseller. Two users. No one else. V1 is a personal power tool.

---

## Feature Summary

### 🎙️ Audio Capture & Transcription
- Captures **system audio + microphone** simultaneously (covers both sides of a virtual meeting)
- Works for **in-person meetings** via microphone only
- Real-time **speech-to-text** via configurable endpoint (local Whisper, cloud API, self-hosted — anything OpenAI-compatible)
- **Pause/resume** recording mid-session
- Full **transcript saved** as a meeting artifact

### 🧠 Real-Time AI Copilot
- Transcript sent to a **reasoning LLM every ~60 seconds**
- LLM is aware of: who the client is, what they use, what you sell, what's been discussed before, and what's been said so far today
- Returns up to **5 suggestions per cycle** (configurable): questions to ask, points to raise, things to note
- **Never repeats** what's already been discussed
- **Waits ~1 minute** before suggesting anything on first-time client calls (needs context before it can help)
- Configurable LLM endpoint — **OpenAI-compatible format** (Ollama, LiteLLM, vLLM, any cloud provider)

### 📋 Live UI
- **Split-pane layout:** notes on the left, AI suggestions on the right
- **Overlay/sidebar mode** — sits alongside Teams/Zoom without stealing focus
- Each suggestion can be:
  - ✓ **Acknowledged** (asked it / already covered)
  - ✗ **Dismissed** (not relevant)
- Suggestions **auto-scroll** upward; notifications are **silent**
- Optional **live transcript view** toggle
- Optional **meeting type** dropdown (client call, internal sync, discovery, etc.)

### 👥 Client Intelligence
- **Client profiles** with technology stack, engagement history, and summarized meeting notes
- **Vendor/product catalog** — your full portfolio, pre-loaded
- Client data **updated organically** from meeting notes with **human-in-the-loop** approval
- **Manual editing** of client profiles outside of meetings
- **Discrepancy handling:** if new info contradicts old notes, it's flagged — user decides what's canonical. Historical notes are **never modified**.

### 📑 Post-Meeting Artifacts
- **Full text transcript** (markdown)
- **Structured meeting summary** — formatted for email copy/paste
- **Action items** — extracted from conversation
- **Client record update suggestions** — proposed changes to client profile, requiring approval
- Artifact generation **driven by meeting type** — client meetings get the full treatment by default

### 🚀 Auto-Prep Mode
- Select a client before starting → app loads their **profile, history, and vendor overlap**
- Get a **pre-meeting briefing** before you hit record

### 💾 Data & Storage
- **All local** — nothing leaves your machine unless you point it at an external API
- **SQLite** for structured data (embedded, zero-config)
- **LanceDB** (or similar) for semantic search across meeting history
- **Flat markdown files** for transcripts and notes (portable, backup-friendly)
- **Dual storage** — searchable in-app AND human-readable on disk

### ⚙️ Configuration
- STT endpoint (URL, API key)
- LLM endpoint (URL, API key, model)
- Max suggestions per cycle (default: 5)
- Meeting type definitions
- Post-meeting action defaults per meeting type
- Transcript storage path

---

## What It Does NOT Do (V1)

| Not in V1 | Why |
|---|---|
| Speaker diarization | Requires meeting platform integration or advanced local models |
| Send emails | Provides formatted text; user copy/pastes |
| Internet research during calls | Scope creep; deferred |
| Multi-user support | Two users only for now |
| Cloud sync | Local-only by design |
| Conversation pattern analysis | V2 feature |
| Auto-modify historical notes | Immutable meeting records by policy |

---

## Complexity at a Glance

| Difficulty | What's Involved |
|---|---|
| 🟢 **Straightforward** | Electron app, SQLite, settings UI, API calls, split-pane layout, post-meeting artifacts |
| 🟡 **Takes work** | Cross-platform audio capture, embedded vector search, context window management, LLM deduplication logic |
| 🔴 **Hard** | Always-on-top overlay without focus steal, audio capture permission handling per OS, intelligent prep mode |
| ⛰️ **Not V1** | Diarization, native meeting bots, fully local reasoning-quality LLM |

---

## One-Line Pitch

> A private, local AI copilot that listens to your meetings, knows your clients, and whispers the right questions in your ear — then writes up the notes when you're done.

---

*See also:*
- *Full requirements: `meeting-copilot-requirements.md`*
- *Ideation transcript: `transcripts/2026-02-14-meeting-copilot-ideation.md`*
