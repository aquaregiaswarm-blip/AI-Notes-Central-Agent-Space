# Meeting Copilot — Detailed Requirements Document
**Version:** 0.1 (Draft)
**Date:** 2026-02-14
**Authors:** Jonathan Gough, Roy Bales (requirements), Clarice (documentation)
**Status:** Ideation / Scoping

---

## 1. Overview

Meeting Copilot is a local desktop application designed to act as a real-time AI-powered assistant during live conversations — whether virtual meetings (Teams, Zoom, WebEx) or in-person discussions. It captures audio, transcribes speech to text, and continuously feeds the transcript to an LLM reasoning model that returns contextual prompts, questions, notes, and suggestions to the user in real time.

The application also serves as a persistent client intelligence system, maintaining historical meeting notes, client profiles, vendor/product catalogs, and summarized engagement history across multiple interactions.

---

## 2. Target Users

- **V1 Users:** Jonathan Gough and Roy Bales only
- **Persona:** High-end technology consultants / Field CTOs acting as mid-to-high-level thought leaders in client engagements
- **Organization type:** Value-added reseller (VAR) covering a broad portfolio of vendors, products, and services
- **Security model (V1):** Two-user internal tool. No multi-tenancy, no external access. Additional security deferred to later versions.

---

## 3. Platform Requirements

| Requirement | Specification |
|---|---|
| **Application type** | Electron desktop app |
| **Supported OS (V1)** | macOS (primary), Windows and Linux (investigate Electron parity) |
| **Deployment model** | Fully local — single executable launch, no separate services to manage |
| **Self-contained** | Electron app must bundle and auto-start all backend components (SQLite, vector DB, local services) on launch. No manual setup. |

### 3.1 Design Principle
> "If we have too many moving pieces for a V1, it'll be delicate. It'll break. We'll never use it." — Jonathan Gough

The application must start and stop as a single unit. One click to open, one click (or window close) to shut down. No external daemons, no manual database starts, no Docker containers for V1.

---

## 4. Core Functional Requirements

### 4.1 Audio Capture & Speech-to-Text (STT)

| ID | Requirement | Priority |
|---|---|---|
| STT-01 | Capture system audio (speakers) and microphone input simultaneously | V1 |
| STT-02 | Work across meeting platforms (Teams, Zoom, WebEx) without integration — system-level audio capture | V1 |
| STT-03 | Work during in-person meetings (microphone-only capture) | V1 |
| STT-04 | Configurable STT endpoint: local model OR external API (cloud or self-hosted) | V1 |
| STT-05 | STT endpoint specification uses **OpenAI-compatible API format** (supporting Ollama, LiteLLM, vLLM, cloud providers) | V1 |
| STT-06 | Ability to **pause and resume** speech-to-text capture during a session | V1 |
| STT-07 | Speaker diarization (identifying who is speaking) | V2 |
| STT-08 | Native meeting platform integration (joining as a participant/agent) | Future |

### 4.2 Real-Time LLM Analysis

| ID | Requirement | Priority |
|---|---|---|
| LLM-01 | Send cumulative transcript to a reasoning LLM at regular intervals (~60 seconds) | V1 |
| LLM-02 | LLM endpoint is configurable, using **OpenAI-compatible API format** (same flexibility as STT) | V1 |
| LLM-03 | LLM context includes: current transcript, client profile, vendor/product catalog, historical meeting summaries for this client | V1 |
| LLM-04 | LLM returns: suggested questions, talking points, contextual notes, relevant observations | V1 |
| LLM-05 | LLM must **not** re-suggest topics already discussed in the current transcript | V1 |
| LLM-06 | LLM must account for already-discussed items to inform deeper/follow-up suggestions | V1 |
| LLM-07 | Maximum **5 suggestions per analysis cycle** (configurable in settings) | V1 |
| LLM-08 | For first-time clients (no prior context), delay suggestions until ~1 minute of conversation has elapsed | V1 |
| LLM-09 | Meeting type (if selected) influences the LLM's prompt strategy and suggestion style | V1 |
| LLM-10 | Conversation pattern detection and behavioral analysis | V2 |
| LLM-11 | External/internet research to augment real-time suggestions | Future |

### 4.3 User Interface

| ID | Requirement | Priority |
|---|---|---|
| UI-01 | **Split-pane layout:** Running notes/transcript on the left, AI suggestions/prompts on the right | V1 |
| UI-02 | **Overlay or sidebar mode** — usable alongside meeting applications without dominating screen | V1 |
| UI-03 | Each suggestion has: **✓ (acknowledge/asked)** and **✗ (dismiss/irrelevant)** actions | V1 |
| UI-04 | Dismissed items are categorized as positive (already covered) or negative (not relevant) | V1 |
| UI-05 | Suggestions auto-scroll upward as new ones arrive; silent notifications only | V1 |
| UI-06 | Optional toggle to view live transcript | V1 |
| UI-07 | Optional dropdown to select **meeting type** (client call, internal sync, discovery, etc.) — not mandatory | V1 |
| UI-08 | Settings panel for: STT endpoint, LLM endpoint, max suggestions per cycle, other preferences | V1 |

### 4.4 Client & Vendor Knowledge Base

| ID | Requirement | Priority |
|---|---|---|
| KB-01 | **Client profiles** with: name, known technology stack, engagement history, summarized notes from all prior meetings | V1 |
| KB-02 | **Vendor/product/services catalog** — pre-populated list of what the organization sells and partners with | V1 |
| KB-03 | Client stack list populated from historical meeting notes (organic updates) | V1 |
| KB-04 | Client stack includes both products they have AND products they don't have (for cross-sell/gap analysis) | V1 |
| KB-05 | Users can **manually view, edit, and update** client attributes outside of meetings | V1 |
| KB-06 | All organic updates from meeting notes are **human-in-the-loop** — flagged for user approval before persisting to client record | V1 |
| KB-07 | When new meeting contradicts earlier meeting data, flag as **discrepancy** in current meeting notes. Let user decide what goes into summary record. **Never modify earlier meeting notes.** | V1 |

### 4.5 Auto-Prep Mode

| ID | Requirement | Priority |
|---|---|---|
| PREP-01 | When a meeting is started with a known client selected, auto-load their profile, historical context, and relevant vendor overlap | V1 |
| PREP-02 | Present a pre-meeting briefing summary before recording begins | V1 |

### 4.6 Post-Meeting Artifacts

| ID | Requirement | Priority |
|---|---|---|
| POST-01 | **Full text transcript** saved as an output artifact | V1 |
| POST-02 | **Meeting summary** — formatted, structured notes suitable for copy/paste into email | V1 |
| POST-03 | **Action items** — extracted from the conversation | V1 |
| POST-04 | **Internal record update** — suggested updates to client profile and notes (human-in-the-loop approval) | V1 |
| POST-05 | Post-meeting actions are **driven by meeting type** (if selected). Client meetings get full artifact generation by default. | V1 |
| POST-06 | No automated email sending. Provide formatted text for manual copy/paste. | V1 |
| POST-07 | Export to email client | Future |

---

## 5. Data Storage Requirements

### 5.1 Storage Architecture

| ID | Requirement | Priority |
|---|---|---|
| DATA-01 | **SQLite** for structured data (client profiles, meeting metadata, vendor catalog, action items) — embedded in Electron, no external DB | V1 |
| DATA-02 | **Vector database** (embedded, e.g., LanceDB or similar) for semantic search over meeting notes and transcripts — auto-starts with app | V1 |
| DATA-03 | **Flat file storage** — transcripts and meeting notes also saved as plain markdown files for portability and backup | V1 |
| DATA-04 | Dual storage: structured/searchable in-app AND flat files on disk | V1 |
| DATA-05 | All data stored locally on user's machine | V1 |

### 5.2 Update & Sync

| ID | Requirement | Priority |
|---|---|---|
| SYNC-01 | App should auto-update but with **human-in-the-loop** approval before applying updates | V1 |

---

## 6. Configuration & Settings

| Setting | Details |
|---|---|
| STT Endpoint | URL + API key, OpenAI-compatible format |
| LLM Endpoint | URL + API key + model name, OpenAI-compatible format |
| Max suggestions per cycle | Default: 5, adjustable |
| Meeting types | Configurable list (client call, internal, discovery, etc.) |
| Default post-meeting actions per type | Configurable |
| Transcript storage location | Local path |

---

## 7. Non-Functional Requirements

| Requirement | Detail |
|---|---|
| **Simplicity** | Single-app experience. No multi-service orchestration for V1. |
| **Resilience** | If LLM or STT endpoint is unreachable, app continues capturing audio and degrades gracefully |
| **Performance** | UI must remain responsive during STT + LLM processing cycles |
| **Privacy** | All data local. No telemetry. No cloud sync in V1. |
| **Portability** | Flat file exports ensure data isn't locked in proprietary formats |

---

## 8. Explicitly Out of Scope (V1)

- Multi-user / multi-tenant support
- Cloud deployment or hosted version
- External internet research during meetings
- Speaker diarization
- Automated email sending
- Conversation pattern detection
- Native meeting platform integrations (Teams bot, Zoom app, etc.)
- Advanced security/auth (SSO, encryption at rest, etc.)

---

## 9. Complexity Assessment

### 🟢 Easy Button — Achievable with standard tooling
- Electron app shell with SQLite (better-sqlite3)
- Settings UI for endpoint configuration
- OpenAI-compatible API calls for STT and LLM
- Split-pane UI with suggestion cards
- Flat file transcript export
- Post-meeting summary generation

### 🟡 Not-So-Easy — Requires careful implementation
- System audio + microphone capture cross-platform (macOS vs Windows audio APIs differ significantly)
- Embedded vector DB with semantic search (LanceDB or Chroma embedded)
- Real-time streaming transcript → periodic LLM analysis loop
- Context window management (cumulative transcripts can exceed token limits)
- Discrepancy detection against historical notes

### 🔴 Hard — Significant engineering effort
- Overlay/sidebar mode that works alongside any app without focus-stealing
- Cross-platform audio capture parity (macOS screen recording permissions, Windows WASAPI, Linux PulseAudio)
- Intelligent deduplication of suggestions across analysis cycles
- Auto-prep mode with semantic retrieval across all historical meetings for a client

### ⛰️ Insurmountable Hill (for V1 — defer)
- Real-time speaker diarization without meeting platform integration
- Native meeting platform agent/bot integration
- Fully local LLM inference with reasoning-model quality (hardware-dependent)

---

## 10. Recommended Tech Stack (V1)

| Component | Recommendation |
|---|---|
| **App shell** | Electron (cross-platform, bundles everything) |
| **Frontend** | React or Svelte (within Electron renderer) |
| **Structured DB** | better-sqlite3 (embedded, zero-config) |
| **Vector DB** | LanceDB (embedded, runs in-process, no server) |
| **Audio capture** | Node.js native modules + platform audio APIs (BlackHole/loopback on macOS, WASAPI on Windows) |
| **STT** | OpenAI Whisper API format — configurable endpoint |
| **LLM** | OpenAI Chat Completions API format — configurable endpoint |
| **File storage** | Local filesystem, markdown format |
| **Auto-update** | electron-updater with confirmation dialog |

---

*Document generated from ideation session on 2026-02-14. See full transcript: `transcripts/2026-02-14-meeting-copilot-ideation.md`*
