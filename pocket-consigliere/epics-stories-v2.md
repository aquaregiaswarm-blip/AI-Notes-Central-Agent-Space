# Pocket Consigliere - Epics & Stories (Hybrid Architecture)

> **Generated from:** Planning session 2026-02-14 + Architecture decision 2026-02-15  
> **Architecture:** Electron + Rust audio addon (napi-rs)  
> **LLM Strategy:** Ollama (local) + Cloud APIs (configurable)  
> **Status:** Draft v2  
> **Version:** v1 scope

---

## Epic Overview

| Epic | Priority | Stories | Est. Complexity |
|------|----------|---------|-----------------|
| 0. Week 1 Proof of Concept | **Critical** | 1 | High |
| 1. Foundation | Must-have | 5 | Medium |
| 2. Audio Capture (Rust Addon) | Must-have | 3 | High |
| 3. Speech-to-Text | Must-have | 2 | Medium |
| 4. Knowledge Base | Must-have | 2 | Medium |
| 5. LLM Analysis Engine | Must-have | 3 | High |
| 6. Live Meeting UI | Must-have | 1 | Medium |
| 7. Auto-Prep Mode | Nice-to-have | 1 | Medium |
| 8. Post-Meeting Artifacts | Nice-to-have | 1 | Low |
| 9. Settings & Config | Must-have | 1 | Low |
| 10. Health & Diagnostics | Must-have | 2 | Medium |
| 11. Packaging & Distribution | Must-have | 1 | Medium |

**Total Stories:** 23 (19 must-have, 4 nice-to-have)

---

## EPIC 0: Week 1 Proof of Concept (CRITICAL PATH)

**Description:** Validate the entire technical stack before full implementation: Rust audio addon (fork Meetily) + Whisper (GPU-accelerated) + Ollama (local LLM) + minimal UI

**Why it matters:** This is a one-way door decision. We MUST prove audio capture works, Ollama latency is acceptable, and the full chain (audio → STT → LLM → UI) is viable.

**Dependencies:** None (this IS the validation)

**Priority:** **CRITICAL — Week 1 deliverable**

**Success Criteria:**
- ✅ Fork Meetily's Rust audio code → compile as napi-rs addon
- ✅ Capture mic + system audio on macOS (30-second test)
- ✅ Whisper.cpp transcribes 60s chunk in <10s (GPU-accelerated)
- ✅ Ollama generates suggestions in <60s (with client context)
- ✅ Minimal UI displays transcript + suggestions

**If this fails:** Pivot to "import audio file" workflow or reconsider architecture

---

### Story 0.1: Full-Stack Proof of Concept

**As a** developer  
**I want** to validate the entire technical chain in Week 1  
**So that** we don't build on unproven assumptions

**Acceptance Criteria:**
- ✅ **Day 1-2:** Fork Meetily's audio module (Rust), compile as `.node` addon via napi-rs
- ✅ **Day 3:** Test audio capture (mic + system audio) on macOS, record 60s test meeting
- ✅ **Day 4:** Whisper.cpp (Metal-accelerated) transcribes 60s chunk in <10s
- ✅ **Day 5:** Ollama (Llama 3.1 8B) analyzes transcript + client context in <60s
- ✅ **Day 6:** Minimal Electron UI displays transcript + 3 suggestions
- ✅ **Day 7:** End-to-end test: simulate 30-min meeting, full cycle (audio → STT → LLM → UI)

**Dependencies:** None (starting point)

**Priority:** v1 (CRITICAL)

**Estimated Complexity:** High (but de-risks everything else)

**Deliverable:** Working prototype OR pivot decision

---

## EPIC 1: Foundation — App Shell & Data Layer

**Description:** Establish the skeleton — Electron app shell, databases (SQLite + vector DB), file storage, IPC layer

**Why it matters:** Ground zero. All features depend on this working foundation.

**Dependencies:** Epic 0 (Week 1 POC validated)

**Priority:** Must-have

---

### Story 1.1: Electron App Scaffold

**As a** developer  
**I want** a production-ready Electron app with React frontend  
**So that** all features have a runtime environment

**Acceptance Criteria:**
- ✅ Electron app launches with blank window on macOS
- ✅ React renderer loads with hot-reload (Vite)
- ✅ Main/renderer IPC works bidirectionally (typed with TypeScript)
- ✅ App starts and stops cleanly (no orphan processes)
- ✅ Project structure: `src/main/`, `src/renderer/`, `src/shared/`, `src/rust-addon/`

**Dependencies:** Story 0.1 (Week 1 POC)

**Priority:** v1

**Estimated Complexity:** Simple (leverages Week 1 POC work)

---

### Story 1.2: SQLite Database Layer

**As a** developer  
**I want** embedded SQLite auto-initializing on launch  
**So that** structured data persists (clients, vendors, products, meetings)

**Acceptance Criteria:**
- ✅ Database file created at `~/Library/Application Support/PocketConsigliere/db.sqlite`
- ✅ Schema migrations run automatically on startup
- ✅ CRUD operations work for all tables (clients, vendors, products, meetings, transcripts)
- ✅ Data persists across app restarts
- ✅ Foreign key constraints enforced

**Dependencies:** Story 1.1 (Electron app scaffold)

**Priority:** v1

**Estimated Complexity:** Moderate

---

### Story 1.3: Vector Database Layer

**As a** developer  
**I want** embedded vector database (LanceDB preferred, ChromaDB alternative)  
**So that** semantic search works on clients, vendors, meeting transcripts

**Acceptance Criteria:**
- ✅ Vector DB initializes on launch (local, no server)
- ✅ Documents can be inserted with embeddings (all-MiniLM-L6-v2, local model)
- ✅ Semantic search returns top-k relevant results (<100ms query time)
- ✅ Integration with SQLite for metadata (vector IDs → SQLite foreign keys)

**Dependencies:** Story 1.1 (Electron app scaffold)

**Priority:** v1

**Estimated Complexity:** Moderate

---

### Story 1.4: Flat File Storage System

**As a** developer  
**I want** structured file storage for transcripts, recordings, briefings  
**So that** large unstructured data doesn't bloat the database

**Acceptance Criteria:**
- ✅ Files stored in `~/Library/Application Support/PocketConsigliere/`
- ✅ Folder structure: `transcripts/`, `recordings/`, `briefings/`
- ✅ File paths stored in SQLite (not file contents)
- ✅ Automatic cleanup of orphaned files (when meeting deleted)

**Dependencies:** Story 1.2 (SQLite database)

**Priority:** v1

**Estimated Complexity:** Simple

---

### Story 1.5: IPC Communication Layer

**As a** developer  
**I want** typed IPC channels for renderer ↔ main communication  
**So that** frontend can trigger backend operations safely

**Acceptance Criteria:**
- ✅ IPC channels defined in `src/shared/ipc-types.ts` (TypeScript)
- ✅ All database operations accessible via IPC
- ✅ Error handling propagates to renderer (Result<T, E> pattern)
- ✅ TypeScript enforces IPC contracts (no runtime type errors)

**Dependencies:** Story 1.1 (Electron app scaffold)

**Priority:** v1

**Estimated Complexity:** Simple

---

## EPIC 2: Audio Capture (Rust Addon)

**Description:** Fork Meetily's Rust audio module → compile as napi-rs addon → capture mic + system audio on macOS

**Why it matters:** Audio capture is the highest risk. Using Meetily's proven code (MIT licensed) de-risks this completely.

**Dependencies:** Epic 0 (Week 1 POC), Epic 1 (Foundation)

**Priority:** Must-have

**Architecture Decision:** Rust addon (not pure Node.js, not full Tauri) for proven audio capture + Node.js comfort zone everywhere else

---

### Story 2.1: Fork Meetily Audio Module → napi-rs Addon

**As a** developer  
**I want** Meetily's Rust audio code compiled as a Node.js addon  
**So that** audio capture is proven and I don't reinvent the wheel

**Acceptance Criteria:**
- ✅ Fork Meetily's audio module (GitHub: Zackriya-Solutions/meeting-minutes, MIT license)
- ✅ Extract audio capture code (uses `cpal` Rust crate for cross-platform audio)
- ✅ Adapt to napi-rs bindings (generate TypeScript types automatically)
- ✅ Compile to `.node` binary (macOS ARM64 + x86_64)
- ✅ Expose simple API: `AudioCapture.start()`, `.stop()`, `.getBuffer()`, `.getStatus()`

**Dependencies:** Story 0.1 (Week 1 POC validated this approach)

**Priority:** v1

**Estimated Complexity:** Moderate (leverages Week 1 POC work)

---

### Story 2.2: System Audio + Mic Capture (macOS)

**As a** user  
**I want** the app to capture system audio (Teams, Zoom, etc.) + my microphone  
**So that** the full conversation is transcribed

**Acceptance Criteria:**
- ✅ BlackHole virtual audio driver integration (user installs separately)
- ✅ System audio captured from active meeting app (configurable input device)
- ✅ Microphone audio captured simultaneously (configurable input device)
- ✅ Audio streams merged with intelligent ducking (prevent feedback)
- ✅ Recording starts/stops on demand (manual or auto-detect future)
- ✅ Audio health check on launch (shows status: working / needs setup)

**Dependencies:** Story 2.1 (Rust audio addon)

**Priority:** v1

**Estimated Complexity:** Moderate (Rust addon handles the hard parts)

---

### Story 2.3: Audio Fallback — Manual Import

**As a** user  
**I want** to import an audio file if live capture fails  
**So that** the app is still useful even if audio permissions/hardware fail

**Acceptance Criteria:**
- ✅ "Import Audio File" button in UI
- ✅ Accepts .wav, .mp3, .m4a, .opus formats
- ✅ Processes imported audio same as live capture (STT → LLM → suggestions)
- ✅ Stored in `recordings/` directory with metadata

**Dependencies:** Story 1.1 (Electron app), Story 2.2 (Audio capture logic)

**Priority:** v1

**Estimated Complexity:** Simple

---

## EPIC 3: Speech-to-Text

**Description:** Transcribe audio chunks using Whisper.cpp (local, GPU-accelerated) or cloud STT (configurable)

**Why it matters:** Can't analyze conversation without transcription

**Dependencies:** Epic 2 (Audio Capture)

**Priority:** Must-have

---

### Story 3.1: Whisper.cpp Integration (GPU-Accelerated)

**As a** user  
**I want** fast, local transcription with Whisper  
**So that** I don't need cloud APIs or pay per-minute

**Acceptance Criteria:**
- ✅ Whisper.cpp binary bundled with app (Metal-enabled for macOS)
- ✅ Transcribe 60s audio chunk in <10s (target: ~5s with GPU)
- ✅ Accuracy >95% for English (test with sample meeting audio)
- ✅ Transcript chunks stored in SQLite + vector DB (for semantic search)
- ✅ Fallback to CPU if GPU unavailable (warn user about latency)

**Dependencies:** Story 2.2 (Audio capture)

**Priority:** v1

**Estimated Complexity:** Moderate

---

### Story 3.2: Cloud STT Fallback (OpenAI-Compatible)

**As a** user  
**I want** to use cloud STT (OpenAI, Groq, etc.) if Whisper is too slow  
**So that** I have flexibility based on my hardware

**Acceptance Criteria:**
- ✅ Configurable STT endpoint (OpenAI-compatible API format)
- ✅ Automatic fallback if Whisper.cpp fails or times out
- ✅ API key stored securely (OS keychain)
- ✅ Cost tracking (tokens used, estimated $)

**Dependencies:** Story 3.1 (Whisper integration)

**Priority:** v1

**Estimated Complexity:** Simple

---

## EPIC 4: Client & Vendor Knowledge Base

**Description:** Store and retrieve client profiles, vendor catalog, product info for contextual suggestions

**Why it matters:** AI needs context about who the client is and what capabilities we can offer

**Dependencies:** Epic 1 (Foundation)

**Priority:** Must-have

---

### Story 4.1: Client Profile Management

**As a** user  
**I want** to create/edit client profiles (name, tech stack, past notes)  
**So that** the AI knows the client's context during meetings

**Acceptance Criteria:**
- ✅ CRUD UI for client profiles
- ✅ Fields: name, industry, tech stack summary, notes, logo (optional)
- ✅ Client data stored in SQLite + vector DB (for semantic search)
- ✅ Client profiles searchable in UI (fuzzy search + semantic search)

**Dependencies:** Story 1.2 (SQLite), Story 1.3 (Vector DB), Story 1.5 (IPC)

**Priority:** v1

**Estimated Complexity:** Moderate

---

### Story 4.2: Vendor & Product Catalog

**As a** user  
**I want** a searchable catalog of vendors and products  
**So that** the AI can suggest relevant solutions during meetings

**Acceptance Criteria:**
- ✅ CRUD UI for vendors and products
- ✅ Fields: vendor name, category, products (many-to-many), descriptions
- ✅ Vector search for semantic lookup ("we need container orchestration" → suggests Kubernetes vendors)
- ✅ Products linked to vendors (foreign keys in SQLite)

**Dependencies:** Story 1.2 (SQLite), Story 1.3 (Vector DB)

**Priority:** v1

**Estimated Complexity:** Moderate

---

## EPIC 5: Real-Time LLM Analysis Engine

**Description:** Analyze transcripts + client context → generate suggestions (questions, notes, research) using Ollama (local) or cloud APIs

**Why it matters:** This is the AI brain — without it, we're just a transcription tool

**Dependencies:** Epic 3 (STT), Epic 4 (Knowledge Base)

**Priority:** Must-have

**Architecture Decision:** Support BOTH Ollama (local, free) and cloud APIs (faster, paid) from day 1

---

### Story 5.1: Ollama Integration (Local LLM)

**As a** user  
**I want** to use Ollama for local LLM inference  
**So that** I don't pay per-token and keep data fully local

**Acceptance Criteria:**
- ✅ Ollama installed and configured (Llama 3.1 8B or Mixtral 8x7B)
- ✅ OpenAI-compatible API client (Ollama exposes OpenAI format)
- ✅ Context window: last 10 min transcript + client profile + vendor catalog
- ✅ Suggestions generated in <60s (target: ~30s on M1/M2 Mac)
- ✅ Timeout handling (if >90s, skip cycle and retry next window)
- ✅ GPU acceleration (Metal on macOS)

**Dependencies:** Story 3.1 (STT), Story 4.1 (Client profiles), Story 4.2 (Vendor catalog)

**Priority:** v1

**Estimated Complexity:** Moderate

---

### Story 5.2: Cloud LLM Integration (OpenAI-Compatible)

**As a** user  
**I want** to use cloud LLMs (Claude, GPT-4, Groq) for faster/better suggestions  
**So that** I can choose speed vs cost vs quality

**Acceptance Criteria:**
- ✅ Configurable LLM endpoint (OpenAI-compatible format)
- ✅ API key stored securely (OS keychain)
- ✅ Same context window as Ollama (last 10 min + client + vendor)
- ✅ Cost tracking (tokens used, estimated $)
- ✅ Automatic failover (if cloud API fails, fall back to Ollama if available)

**Dependencies:** Story 5.1 (Ollama integration)

**Priority:** v1

**Estimated Complexity:** Simple (reuses Ollama client code)

---

### Story 5.3: Sliding Window Context Management

**As a** user  
**I want** the AI to maintain context without hitting token limits  
**So that** 2-hour meetings don't break the app

**Acceptance Criteria:**
- ✅ Sliding window: last 10 minutes full transcript + summarized earlier context
- ✅ Token counter UI (show remaining capacity in real-time)
- ✅ Automatic summarization when window slides (compress old context)
- ✅ Max meeting length: 2 hours (graceful degradation beyond)
- ✅ Test with 2-hour meeting simulation (no crashes, quality maintained)

**Dependencies:** Story 5.1 (Ollama), Story 5.2 (Cloud LLM)

**Priority:** v1

**Estimated Complexity:** Moderate

---

## EPIC 6: Live Meeting UI

**Description:** Split-pane UI showing transcript on left, suggestions on right, with triage actions

**Why it matters:** Users need to see and interact with suggestions in real-time

**Dependencies:** Epic 5 (LLM Analysis Engine)

**Priority:** Must-have

---

### Story 6.1: Split-Pane Meeting View

**As a** user  
**I want** a split-pane UI (transcript | suggestions) during meetings  
**So that** I can see what's being said and what the AI recommends

**Acceptance Criteria:**
- ✅ Left pane: live transcript (auto-scrolling, last 50 messages visible)
- ✅ Right pane: suggestions (questions, notes, research, actions)
- ✅ Triage actions: ✓ (acknowledge), ✗ (dismiss), copy to clipboard
- ✅ Suggestions update every 60-90 seconds (configurable)
- ✅ Progress indicator during LLM analysis ("Analyzing... 15s remaining")
- ✅ UI doesn't block or freeze during analysis (async IPC)
- ✅ Suggestion aging (dim/collapse suggestions after 5 minutes if not dismissed)

**Dependencies:** Story 5.3 (LLM analysis with context management)

**Priority:** v1

**Estimated Complexity:** Moderate

---

## EPIC 7: Auto-Prep Mode

**Description:** Before a meeting, generate a briefing (client context, past meeting summaries, suggested questions)

**Why it matters:** Helps user prepare for meetings, reduces cold-start anxiety

**Dependencies:** Epic 4 (Knowledge Base), Epic 5 (LLM Analysis Engine)

**Priority:** Nice-to-have (v1 if time permits, otherwise v2)

---

### Story 7.1: Pre-Meeting Briefing

**As a** user  
**I want** a briefing generated before a meeting starts  
**So that** I know the client's context and suggested talking points

**Acceptance Criteria:**
- ✅ User selects client + meeting type (discovery, demo, technical deep-dive)
- ✅ AI generates briefing: client summary, past meetings recap, suggested questions
- ✅ Briefing saved to `briefings/` directory and displayed in UI
- ✅ Briefing includes vendor/product recommendations (based on client tech stack)

**Dependencies:** Story 4.1 (Client profiles), Story 4.2 (Vendor catalog), Story 5.1 (LLM)

**Priority:** v1 (nice-to-have)

**Estimated Complexity:** Moderate

---

## EPIC 8: Post-Meeting Artifacts

**Description:** After a meeting, generate summary, action items, follow-up email draft

**Why it matters:** Closes the loop — meeting intel becomes actionable documentation

**Dependencies:** Epic 5 (LLM Analysis Engine)

**Priority:** Nice-to-have (v1 if time permits, otherwise v2)

---

### Story 8.1: Artifact Generation

**As a** user  
**I want** a meeting summary + action items + follow-up email draft after a meeting  
**So that** I can quickly document outcomes and send follow-ups

**Acceptance Criteria:**
- ✅ Generate meeting summary (key points, decisions, next steps)
- ✅ Extract action items (assigned to whom, due when)
- ✅ Draft follow-up email (customizable before sending, not auto-sent)
- ✅ Artifacts saved to `briefings/` directory and stored in database

**Dependencies:** Story 5.1 (LLM analysis)

**Priority:** v1 (nice-to-have)

**Estimated Complexity:** Simple

---

## EPIC 9: Settings & Configuration

**Description:** User-configurable settings (API keys, audio devices, LLM model, suggestion frequency)

**Why it matters:** Users need control over how the app behaves

**Dependencies:** Epic 1 (Foundation)

**Priority:** Must-have

---

### Story 9.1: Settings UI

**As a** user  
**I want** a settings panel where I can configure API keys, audio, and LLM preferences  
**So that** the app works with my accounts and devices

**Acceptance Criteria:**
- ✅ Settings UI accessible from menu bar
- ✅ Tabs: General, Audio, STT, LLM, Advanced
- ✅ **General:** App preferences (auto-start, update checks)
- ✅ **Audio:** Input devices (mic + system audio), BlackHole setup instructions
- ✅ **STT:** Whisper (local) vs cloud (OpenAI-compatible endpoint + API key)
- ✅ **LLM:** Ollama (local) vs cloud (endpoint + API key), model selection
- ✅ **Advanced:** Suggestion frequency, context window size, verbose logging
- ✅ Settings persist across restarts (stored in SQLite)
- ✅ Validation for required fields (API keys, audio devices)

**Dependencies:** Story 1.5 (IPC layer)

**Priority:** v1

**Estimated Complexity:** Simple

---

## EPIC 10: Health & Diagnostics (NEW)

**Description:** System health dashboard, safe mode launch, graceful degradation, operational safeguards

**Why it matters:** Two-person team needs fast diagnosis when things break during live client calls

**Dependencies:** All other epics (builds on top)

**Priority:** Must-have

**Rationale:** Operational reality — if we can't debug it at 2 AM, it's not production-ready

---

### Story 10.1: System Health Dashboard

**As a** user  
**I want** a health dashboard showing status of all subsystems  
**So that** I can diagnose issues quickly without digging into logs

**Acceptance Criteria:**
- ✅ Health panel in settings or menu bar
- ✅ Shows status: Audio (✓ working / ⚠ needs setup / ✗ failed)
- ✅ Shows status: STT (✓ working / ⚠ slow / ✗ failed)
- ✅ Shows status: LLM (✓ working / ⚠ slow / ✗ failed)
- ✅ Shows status: Database (✓ healthy / ⚠ slow / ✗ corrupted)
- ✅ Shows latency metrics (avg STT time, avg LLM time)
- ✅ "Test Audio" button (records 5s, transcribes, shows result)
- ✅ "Test LLM" button (sends test prompt, shows response + timing)

**Dependencies:** All previous epics (integrates with every subsystem)

**Priority:** v1

**Estimated Complexity:** Moderate

---

### Story 10.2: Graceful Degradation & Safe Mode

**As a** user  
**I want** the app to degrade gracefully when components fail  
**So that** I can still use partial functionality instead of total failure

**Acceptance Criteria:**
- ✅ If audio capture fails → show warning, enable "Import Audio File" workflow
- ✅ If STT fails → queue transcription, process when available (or use fallback)
- ✅ If LLM times out (>90s) → skip cycle, retry next window
- ✅ If vector DB fails → disable semantic search, app still works (just slower)
- ✅ **Safe mode launch:** `--safe-mode` CLI flag disables all integrations (audio, STT, LLM)
- ✅ Safe mode UI: "Running in safe mode. Only database operations available."

**Dependencies:** Story 10.1 (Health dashboard)

**Priority:** v1

**Estimated Complexity:** Moderate

---

## EPIC 11: Packaging & Distribution

**Description:** Build macOS app bundle, code signing, notarization, auto-updater

**Why it matters:** Users need to install and run the app easily

**Dependencies:** All other epics (final packaging step)

**Priority:** Must-have

---

### Story 11.1: macOS App Bundle

**As a** user  
**I want** a downloadable .dmg or .app bundle  
**So that** I can install and run Pocket Consigliere on my Mac

**Acceptance Criteria:**
- ✅ App builds into signed .app bundle (ARM64 + x86_64 universal binary)
- ✅ DMG installer created with electron-builder
- ✅ Rust addon (`.node` file) bundled correctly for both architectures
- ✅ Whisper.cpp binary bundled (Metal-enabled)
- ✅ Code signing + notarization (for macOS Gatekeeper)
- ✅ App icon and branding applied
- ✅ Auto-updater configured (Electron's built-in updater, checks GitHub releases)

**Dependencies:** All previous epics

**Priority:** v1

**Estimated Complexity:** Moderate

---

## Critical Path to MVP

These stories must be completed for a minimal viable product:

**Week 1:**
1. ✅ Story 0.1 - Full-Stack Proof of Concept (audio → STT → LLM → UI)

**Foundation (Week 2):**
2. Story 1.1 - Electron App Scaffold
3. Story 1.2 - SQLite Database Layer
4. Story 1.5 - IPC Communication Layer

**Audio & STT (Week 3-4):**
5. Story 2.1 - Fork Meetily Audio → napi-rs Addon
6. Story 2.2 - System Audio + Mic Capture (macOS)
7. Story 3.1 - Whisper.cpp Integration (GPU-Accelerated)

**Knowledge Base (Week 4-5):**
8. Story 4.1 - Client Profile Management
9. Story 4.2 - Vendor & Product Catalog

**LLM Engine (Week 5-6):**
10. Story 5.1 - Ollama Integration
11. Story 5.2 - Cloud LLM Integration
12. Story 5.3 - Sliding Window Context Management

**UI & Polish (Week 6-7):**
13. Story 6.1 - Split-Pane Meeting View
14. Story 9.1 - Settings UI
15. Story 10.1 - System Health Dashboard
16. Story 10.2 - Graceful Degradation & Safe Mode

**Ship (Week 7-8):**
17. Story 11.1 - macOS App Bundle

**Total MVP Stories:** 17 of 23

---

## Deferred to v2

These stories are explicitly out of scope for v1:

- **Story 7.1 - Pre-Meeting Briefing** (nice-to-have for v1, guaranteed for v2)
- **Story 8.1 - Post-Meeting Artifacts** (nice-to-have for v1, guaranteed for v2)
- **Story 2.3 - Audio Fallback (Manual Import)** (v1 nice-to-have, easy to add)
- **Story 3.2 - Cloud STT Fallback** (v1 nice-to-have, easy to add)
- **Speaker diarization** (v2+ only)
- **Multi-user / team features** (v2+ only)
- **External research / web scraping** (v2+ only)
- **Native meeting integration (Teams, Zoom plugins)** (v2+ only)
- **Windows/Linux support** (v2+ only, requires Rust addon cross-compilation)

---

## Story Count by Priority

- **v1 (Critical - Week 1 POC):** 1 story
- **v1 (Must-have):** 16 stories
- **v1 (Nice-to-have):** 6 stories
- **v2+:** Various deferred features (not yet scoped as stories)

---

## Technical Stack (Updated)

**Frontend:**
- Electron (app shell)
- React + TypeScript (UI)
- Vite (dev server + hot reload)

**Backend:**
- Node.js + TypeScript (business logic)
- **Rust addon (napi-rs)** — audio capture only (forked from Meetily)
- SQLite (structured data)
- LanceDB or ChromaDB (vector embeddings)

**Audio:**
- **Rust addon:** `cpal` crate (cross-platform audio I/O)
- BlackHole (macOS virtual audio driver, user-installed)
- Intelligent ducking, clipping prevention (from Meetily)

**STT:**
- **Whisper.cpp** (local, Metal-accelerated on macOS)
- OpenAI-compatible cloud STT (fallback)

**LLM:**
- **Ollama** (local, Llama 3.1 8B or Mixtral 8x7B)
- OpenAI-compatible cloud APIs (Claude, GPT-4, Groq)

**Embeddings:**
- `all-MiniLM-L6-v2` (local, fast, small)

**Platform:**
- macOS-only for v1 (ARM64 + x86_64 universal binary)
- Windows/Linux in v2+ (Rust addon cross-compilation required)

**Deployment:**
- Self-contained .app bundle (no server dependencies)
- Auto-updater (checks GitHub releases)

**Privacy:**
- Fully local (except cloud API calls if configured)
- No telemetry

---

## Performance Targets (Realistic)

| Metric | Target | Acceptable | Timeout |
|--------|--------|------------|---------|
| **UI responsiveness** | <100ms | <200ms | N/A |
| **STT (Whisper, GPU)** | <5s for 60s audio | <10s | 30s |
| **LLM (Ollama)** | <30s analysis | <60s | 90s |
| **LLM (Cloud)** | <10s analysis | <20s | 60s |
| **Total cycle time** | <45s | <90s | 120s |
| **Max meeting length** | 2 hours | 3 hours | 4 hours (degraded) |

---

## Architecture Decisions (Rationale)

### Why Electron + Rust Addon (Hybrid)?
- ✅ Audio risk mitigated (Meetily's proven code)
- ✅ Node.js comfort zone for business logic (fast iteration)
- ✅ Reversible (can replace addon with Node-native in v2 if desired)
- ✅ AI-assisted debugging (Rust errors are patterns AI recognizes)
- ✅ Better Linux support (no WebKitGTK dependency like Tauri)

### Why Ollama + Cloud APIs (Both)?
- ✅ Privacy (local inference, no per-token costs)
- ✅ Flexibility (users choose speed vs cost vs quality)
- ✅ Reliability (cloud fallback if Ollama fails/slow)

### Why Week 1 POC?
- ✅ De-risks audio capture (make-or-break)
- ✅ Validates Ollama latency (<60s acceptable?)
- ✅ Proves full chain before building everything else
- ✅ Go/no-go decision point (pivot if POC fails)

---

## Related Documents

- [Requirements (updated)](pocket-consigliere-requirements-v2.md)
- [Implementation Plan (updated)](pocket-consigliere-implementation-plan-v2.md)
- [Risk Considerations](risk-considerations.md)
- [Market Comparison](market-compare-and-contrast.md)
- [Original Ideation Transcript](transcripts/2026-02-14-pocket-consigliere-ideation.md)
