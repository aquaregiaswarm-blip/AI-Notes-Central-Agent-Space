# Pocket Consigliere — V1 Implementation Plan
**Version:** 0.1  
**Date:** 2026-02-14  
**Status:** Planning  

---

## Overview

This document breaks down the Pocket Consigliere V1 epics and stories into concrete tasks, subtasks, test plans, and dependency maps. Every story includes:
- **Tasks & Subtasks** — granular work items
- **Test Plans** — unit, functional, integration, and E2E tests
- **Dependencies & Blockers** — what must be done first
- **Acceptance Criteria** — how we know it's done

---

## Build Order (Dependency Map)

```
EPIC 1: Foundation ──────────────────────┐
    ↓                                    │
EPIC 2: Audio & STT ─────────────────┐   │
    ↓                                │   │
EPIC 3: Knowledge Base               │   │
    ↓                                │   │
EPIC 4: LLM Analysis Engine ←────────┤   │
    ↓                                    │
EPIC 5: Live Meeting UI ←───────────────┘
    ↓
EPIC 6: Auto-Prep Mode
    ↓
EPIC 7: Post-Meeting Artifacts
    ↓
EPIC 8: Settings & Config
    ↓
EPIC 9: Packaging & Distribution
```

---

# EPIC 1: Foundation — App Shell & Data Layer

**Goal:** Establish the skeleton — app shell, databases, IPC, file storage  
**Dependencies:** None (ground zero)

---

## Story 1.1: Electron App Scaffold

**As a** developer, **I want** a working Electron app with React frontend, **so that** all features have a runtime.

### Tasks
- [ ] **1.1.1** Initialize Electron project with electron-forge or electron-builder
- [ ] **1.1.2** Configure TypeScript for main + renderer processes
- [ ] **1.1.3** Set up React in renderer with hot-reload (Vite or Webpack)
- [ ] **1.1.4** Create basic IPC bridge between main and renderer
- [ ] **1.1.5** Implement clean app lifecycle (start/stop without orphan processes)
- [ ] **1.1.6** Set up project structure: `src/main/`, `src/renderer/`, `src/shared/`

### Subtasks (1.1.1 — Initialize Electron)
- Install Electron + TypeScript dependencies
- Configure `package.json` scripts (dev, build, package)
- Create `src/main/index.ts` entry point
- Create `src/renderer/index.html` + `index.tsx`

### Subtasks (1.1.4 — IPC Bridge)
- Define typed IPC channels in `src/shared/ipc-types.ts`
- Implement `preload.ts` with contextBridge
- Create main-process IPC handlers
- Test bidirectional communication

### Test Plans

**Unit Tests:**
- Main process launches without error
- Renderer process loads
- IPC sends and receives messages correctly
- TypeScript types enforce IPC contract

**Functional Tests:**
- App window opens on launch
- Hot-reload triggers on file change
- App quits cleanly without orphan processes

**Integration Tests:**
- Main ↔ Renderer communication via IPC
- Multiple IPC channels work simultaneously

**E2E Tests:**
- Launch app → verify window visible → send test IPC call → verify response → quit app

### Dependencies & Blockers
- **Dependencies:** None
- **Blockers:** None

### Acceptance Criteria
- ✅ Electron app launches with blank window on macOS
- ✅ React renderer loads with hot-reload
- ✅ Main/renderer IPC works bidirectionally
- ✅ App starts and stops cleanly (no orphan processes)
- ✅ TypeScript configured for both processes

---

## Story 1.2: SQLite Database Layer

**As a** developer, **I want** embedded SQLite auto-initializing on launch, **so that** structured data persists.

### Tasks
- [ ] **1.2.1** Integrate `better-sqlite3` in main process
- [ ] **1.2.2** Create database initialization module (`src/main/db/init.ts`)
- [ ] **1.2.3** Implement schema migration system (versioned, auto-run on startup)
- [ ] **1.2.4** Define initial schema (clients, vendors, products, meetings, notes, action_items, suggestions)
- [ ] **1.2.5** Create CRUD utilities for each table
- [ ] **1.2.6** Ensure DB opens on start, closes on quit

### Subtasks (1.2.3 — Migration System)
- Create `migrations/` folder with `001_initial.sql`, etc.
- Track applied migrations in `schema_version` table
- Run pending migrations on startup
- Log migration success/failure

### Subtasks (1.2.4 — Schema Design)
- Design `clients` table (id, name, stack_summary, notes_summary, created_at, updated_at)
- Design `vendors` table (id, name, category, description)
- Design `products` table (id, vendor_id, name, category)
- Design `meetings` table (id, client_id, type, date, transcript_path, summary)
- Design `meeting_notes` table (id, meeting_id, note_text, timestamp)
- Design `action_items` table (id, meeting_id, item_text, status)
- Design `suggestions` table (id, meeting_id, suggestion_text, acknowledged, dismissed)

### Test Plans

**Unit Tests:**
- Database file created in correct location
- Schema migrations run successfully
- CRUD operations for each table work correctly
- Foreign key constraints enforced

**Functional Tests:**
- Database initializes on first launch
- Data persists across app restarts
- Migration system handles out-of-order runs gracefully

**Integration Tests:**
- Multiple tables can be queried in single transaction
- IPC layer can trigger DB operations from renderer

**E2E Tests:**
- Launch app → insert test client → restart app → verify client still exists

### Dependencies & Blockers
- **Dependencies:** Story 1.1 (app shell)
- **Blockers:** None

### Acceptance Criteria
- ✅ `better-sqlite3` integrated
- ✅ Database file created in app user-data directory on first launch
- ✅ Migration system auto-runs on startup
- ✅ All tables created correctly
- ✅ DB opens/closes cleanly with app lifecycle

---

## Story 1.3: Vector Database Layer

**As a** developer, **I want** embedded vector DB for semantic search, **so that** LLM can retrieve context.

### Tasks
- [ ] **1.3.1** Integrate LanceDB (or alternative: Chroma embedded mode)
- [ ] **1.3.2** Initialize vector store on app launch alongside SQLite
- [ ] **1.3.3** Create collections: `meeting_transcripts`, `client_notes`, `meeting_summaries`
- [ ] **1.3.4** Configure embedding generation (local model or API endpoint — must be configurable)
- [ ] **1.3.5** Implement insert, query, delete operations
- [ ] **1.3.6** Ensure no separate server process (in-process only)

### Subtasks (1.3.4 — Embedding Config)
- Support OpenAI embeddings API format
- Support local embedding model (e.g., all-MiniLM-L6-v2 via ONNX)
- Make endpoint/model configurable in settings

### Subtasks (1.3.5 — Vector Operations)
- `insertDocument(collection, text, metadata)` → vectorizes and stores
- `searchSimilar(collection, query, topK)` → returns most relevant docs
- `deleteDocument(collection, id)` → removes from index

### Test Plans

**Unit Tests:**
- Vector DB initializes without error
- Collections created correctly
- Insert operation stores embeddings
- Query returns results ranked by similarity
- Delete operation removes documents

**Functional Tests:**
- Embedding generation works (local or API)
- Search results are semantically relevant
- Vector store persists across restarts

**Integration Tests:**
- SQLite references (meeting IDs) link to vector store documents
- IPC can trigger vector search from renderer

**E2E Tests:**
- Launch app → insert meeting transcript → search for relevant passage → verify correct result

### Dependencies & Blockers
- **Dependencies:** Story 1.1 (app shell)
- **Blockers:** None (can run in parallel with 1.2)

### Acceptance Criteria
- ✅ LanceDB integrated in main process
- ✅ Vector store initializes automatically on launch
- ✅ Collections created for transcripts, notes, summaries
- ✅ Embedding generation configurable (local or API)
- ✅ Insert, query, delete operations working
- ✅ No separate server process

---

## Story 1.4: Flat File Storage System

**As a** developer, **I want** transcripts saved as markdown, **so that** data is portable.

### Tasks
- [ ] **1.4.1** Create file storage module (`src/main/storage/files.ts`)
- [ ] **1.4.2** Implement configurable root directory (default: `~/MeetingCopilot/`)
- [ ] **1.4.3** Define directory structure: `{root}/{client_name}/{date}-{meeting_title}/`
- [ ] **1.4.4** Create atomic file write utility (write to temp, rename)
- [ ] **1.4.5** Store file paths in SQLite for cross-reference
- [ ] **1.4.6** Generate markdown files: `transcript.md`, `summary.md`, `action-items.md`

### Subtasks (1.4.4 — Atomic Writes)
- Write to `{filename}.tmp`
- Verify write success
- Rename to final filename
- Handle errors gracefully (cleanup temp files)

### Test Plans

**Unit Tests:**
- File storage module creates directories correctly
- Atomic write prevents corruption
- File paths stored in SQLite correctly

**Functional Tests:**
- Markdown files generated with correct content
- Files organized in correct directory structure
- Duplicate meeting names handled gracefully

**Integration Tests:**
- SQLite `meetings` table references file paths correctly
- File changes reflected in database metadata

**E2E Tests:**
- Complete meeting → verify transcript, summary, and action-items files created → verify readable markdown

### Dependencies & Blockers
- **Dependencies:** Story 1.2 (SQLite for path storage)
- **Blockers:** None

### Acceptance Criteria
- ✅ Configurable root directory
- ✅ Directory structure: `{client}/{date}-{title}/`
- ✅ Each meeting produces: `transcript.md`, `summary.md`, `action-items.md`
- ✅ Atomic writes prevent corruption
- ✅ File paths stored in SQLite

---

## Story 1.5: IPC Communication Layer

**As a** developer, **I want** typed IPC layer, **so that** UI can request data safely.

### Tasks
- [ ] **1.5.1** Define IPC channel types in `src/shared/ipc-types.ts`
- [ ] **1.5.2** Implement request/response pattern with error handling
- [ ] **1.5.3** Implement event streaming pattern for real-time updates
- [ ] **1.5.4** Create IPC handlers for: DB CRUD, audio control, LLM requests, settings
- [ ] **1.5.5** Add TypeScript validation for all messages

### Subtasks (1.5.2 — Request/Response)
- Define `IPCRequest<T>` and `IPCResponse<T>` types
- Implement timeout handling (default: 5s)
- Return typed errors on failure

### Subtasks (1.5.3 — Event Streaming)
- Define `IPCEvent<T>` type
- Allow renderer to subscribe to event streams
- Implement `transcriptUpdate`, `suggestionReceived`, `audioStateChanged` events

### Test Plans

**Unit Tests:**
- IPC types enforce correct payloads
- Request/response pattern handles errors
- Event streaming delivers messages

**Functional Tests:**
- Timeout triggers on slow handler
- Multiple subscribers receive same event
- IPC handles concurrent requests

**Integration Tests:**
- All DB operations accessible via IPC
- Audio control triggers from renderer
- LLM requests routed correctly

**E2E Tests:**
- Renderer requests client list → main returns data → renderer displays

### Dependencies & Blockers
- **Dependencies:** Story 1.1 (app shell), Story 1.2 (database)
- **Blockers:** None

### Acceptance Criteria
- ✅ Typed IPC channels for all operations
- ✅ Request/response with error handling
- ✅ Event streaming for real-time updates
- ✅ TypeScript validation enforced

---

# EPIC 2: Audio Capture & Speech-to-Text

**Goal:** Capture system audio + microphone, send to STT endpoint  
**Dependencies:** Story 1.1 (app shell), Story 1.5 (IPC layer)

---

## Story 2.1: System Audio Capture (macOS)

**As a** user, **I want** the app to capture meeting audio, **so that** it can transcribe conversations.

### Tasks
- [ ] **2.1.1** Research macOS audio capture APIs (AVFoundation, BlackHole loopback)
- [ ] **2.1.2** Implement audio input selection (system audio + microphone)
- [ ] **2.1.3** Request microphone permissions (handle rejection gracefully)
- [ ] **2.1.4** Stream audio data to buffer
- [ ] **2.1.5** Implement pause/resume controls
- [ ] **2.1.6** Export audio to PCM/WAV format for STT

### Subtasks (2.1.2 — Audio Input)
- Enumerate available audio devices
- Allow user to select microphone device
- Allow user to select system audio loopback (BlackHole or similar)
- Mix both streams into single buffer

### Test Plans

**Unit Tests:**
- Audio device enumeration returns list
- Permission request handled correctly
- Audio buffer fills without gaps

**Functional Tests:**
- System audio captured during test meeting
- Microphone captured correctly
- Pause/resume stops and starts capture
- Audio export produces valid WAV file

**Integration Tests:**
- IPC triggers audio start/stop from renderer
- Audio buffer streams to STT module

**E2E Tests:**
- Launch app → start meeting → capture audio → stop → verify WAV file created

### Dependencies & Blockers
- **Dependencies:** Story 1.1, 1.5
- **Blockers:** macOS microphone permissions must be granted

### Acceptance Criteria
- ✅ Captures system audio + microphone simultaneously
- ✅ Works across Teams, Zoom, WebEx (system-level capture)
- ✅ Pause/resume controls work
- ✅ Exports audio to WAV format

---

## Story 2.2: Speech-to-Text Integration

**As a** user, **I want** audio transcribed in real time, **so that** the AI can analyze it.

### Tasks
- [ ] **2.2.1** Create STT client module (`src/main/stt/client.ts`)
- [ ] **2.2.2** Support OpenAI-compatible API format (Whisper endpoint)
- [ ] **2.2.3** Configure endpoint URL + API key in settings
- [ ] **2.2.4** Send audio chunks to STT endpoint every ~5-10 seconds
- [ ] **2.2.5** Stream transcript updates to renderer via IPC
- [ ] **2.2.6** Handle STT failures gracefully (retry, fallback, error display)

### Subtasks (2.2.4 — Audio Chunking)
- Buffer audio in 5-10 second windows
- Send chunk to STT endpoint
- Append new text to cumulative transcript

### Test Plans

**Unit Tests:**
- STT client sends correct API request
- Response parsed correctly
- Retry logic works on failure

**Functional Tests:**
- Audio chunked and sent at correct intervals
- Transcript updates displayed in real time
- Failures handled without crashing app

**Integration Tests:**
- Audio capture → STT → transcript display
- IPC delivers transcript updates to renderer

**E2E Tests:**
- Start meeting → speak into microphone → verify transcript appears in UI

### Dependencies & Blockers
- **Dependencies:** Story 2.1 (audio capture)
- **Blockers:** STT endpoint must be configured and reachable

### Acceptance Criteria
- ✅ OpenAI-compatible API format
- ✅ Configurable endpoint + API key
- ✅ Sends audio every 5-10 seconds
- ✅ Streams transcript to UI
- ✅ Handles failures gracefully

---

# EPIC 3: Client & Vendor Knowledge Base

**Goal:** Store and retrieve client profiles and vendor catalog  
**Dependencies:** Story 1.2 (SQLite), Story 1.3 (vector DB)

---

## Story 3.1: Client Profile Management

**As a** user, **I want** to maintain client profiles, **so that** the AI has context during meetings.

### Tasks
- [ ] **3.1.1** Create client CRUD UI in renderer
- [ ] **3.1.2** Implement IPC handlers for client operations
- [ ] **3.1.3** Store client data in SQLite
- [ ] **3.1.4** Support manual editing of client profiles
- [ ] **3.1.5** Display client list with search/filter
- [ ] **3.1.6** Link client profiles to meetings

### Test Plans

**Unit Tests:**
- Client CRUD operations work
- Search/filter returns correct results
- Client-meeting links stored correctly

**Functional Tests:**
- Create client → edit → delete → verify persistence
- Search finds clients by name
- Meeting linked to correct client

**Integration Tests:**
- Client data flows from UI → IPC → SQLite
- Vector DB indexes client notes

**E2E Tests:**
- Add client "Delta Airlines" → edit tech stack → create meeting → verify client linked

### Dependencies & Blockers
- **Dependencies:** Story 1.2, 1.5
- **Blockers:** None

### Acceptance Criteria
- ✅ Client CRUD UI
- ✅ Manual editing supported
- ✅ Search/filter works
- ✅ Clients linked to meetings

---

## Story 3.2: Vendor & Product Catalog

**As a** user, **I want** a pre-populated vendor catalog, **so that** the AI knows what we sell.

### Tasks
- [ ] **3.2.1** Create vendor/product CRUD UI
- [ ] **3.2.2** Import initial vendor list (CSV or JSON)
- [ ] **3.2.3** Store vendors/products in SQLite
- [ ] **3.2.4** Support bulk import/export
- [ ] **3.2.5** Link products to vendors

### Test Plans

**Unit Tests:**
- Vendor CRUD operations work
- Product links to correct vendor
- Import parses CSV correctly

**Functional Tests:**
- Import vendor list → verify in UI
- Add product → link to vendor → verify

**Integration Tests:**
- Vendor data flows to LLM context

**E2E Tests:**
- Import vendors → create meeting → verify LLM has vendor context

### Dependencies & Blockers
- **Dependencies:** Story 1.2, 1.5
- **Blockers:** None

### Acceptance Criteria
- ✅ Vendor/product CRUD UI
- ✅ CSV import/export
- ✅ Products linked to vendors

---

# EPIC 4: Real-Time LLM Analysis Engine

**Goal:** Send transcript to LLM every ~60s, return suggestions  
**Dependencies:** Story 2.2 (STT), Story 3.1 (clients), Story 3.2 (vendors)

---

## Story 4.1: LLM Client Integration

**As a** user, **I want** the AI to analyze the meeting, **so that** I get real-time suggestions.

### Tasks
- [ ] **4.1.1** Create LLM client module (`src/main/llm/client.ts`)
- [ ] **4.1.2** Support OpenAI-compatible API format
- [ ] **4.1.3** Configure endpoint, API key, model in settings
- [ ] **4.1.4** Send cumulative transcript every ~60 seconds
- [ ] **4.1.5** Include client profile + vendor catalog in context
- [ ] **4.1.6** Parse LLM response (questions, notes, suggestions)
- [ ] **4.1.7** Limit suggestions to max 5 per cycle (configurable)

### Subtasks (4.1.4 — Context Assembly)
- Build prompt with: transcript, client stack, vendor catalog, historical notes
- Manage token limits (truncate transcript if exceeds window)
- Include instruction: "Do not repeat previously discussed topics"

### Subtasks (4.1.6 — Response Parsing)
- Expect JSON response: `{questions: [], notes: [], suggestions: []}`
- Validate response structure
- Handle malformed responses gracefully

### Test Plans

**Unit Tests:**
- LLM client sends correct API request
- Context assembly includes all required data
- Response parsed correctly
- Suggestion limit enforced

**Functional Tests:**
- Transcript sent every ~60 seconds
- Suggestions appear in UI
- Deduplication works (no repeated suggestions)

**Integration Tests:**
- STT → LLM → UI flow
- Client + vendor context included

**E2E Tests:**
- Start meeting → transcript generates → LLM returns suggestions → suggestions displayed

### Dependencies & Blockers
- **Dependencies:** Story 2.2, 3.1, 3.2
- **Blockers:** LLM endpoint must be configured and reachable

### Acceptance Criteria
- ✅ OpenAI-compatible format
- ✅ Configurable endpoint + model
- ✅ Sends transcript every ~60s
- ✅ Includes client + vendor context
- ✅ Returns max 5 suggestions per cycle
- ✅ No repeated suggestions

---

# EPIC 5: Live Meeting UI

**Goal:** Display transcript + suggestions, allow user to acknowledge/dismiss  
**Dependencies:** Story 1.5 (IPC), Story 2.2 (STT), Story 4.1 (LLM)

---

## Story 5.1: Split-Pane Meeting View

**As a** user, **I want** a split-pane UI, **so that** I can see transcript and suggestions simultaneously.

### Tasks
- [ ] **5.1.1** Design split-pane layout (left: transcript, right: suggestions)
- [ ] **5.1.2** Implement transcript display (auto-scroll, search)
- [ ] **5.1.3** Implement suggestion cards with ✓/✗ actions
- [ ] **5.1.4** Add meeting type dropdown (optional)
- [ ] **5.1.5** Add pause/resume recording button
- [ ] **5.1.6** Display real-time status (recording, processing, idle)

### Test Plans

**Unit Tests:**
- UI components render correctly
- Suggestion actions trigger IPC calls

**Functional Tests:**
- Transcript auto-scrolls as new text arrives
- Suggestions appear in real time
- Acknowledge/dismiss updates state

**Integration Tests:**
- IPC events update UI
- User actions flow back to main process

**E2E Tests:**
- Start meeting → verify UI displays transcript → verify suggestions appear → acknowledge suggestion → verify removed from view

### Dependencies & Blockers
- **Dependencies:** Story 2.2, 4.1
- **Blockers:** None

### Acceptance Criteria
- ✅ Split-pane layout
- ✅ Transcript auto-scrolls
- ✅ Suggestions display in real time
- ✅ Acknowledge/dismiss actions work
- ✅ Optional meeting type dropdown

---

# EPIC 6: Auto-Prep Mode

**Goal:** Load client context before meeting starts  
**Dependencies:** Story 3.1 (clients), Story 1.3 (vector DB)

---

## Story 6.1: Pre-Meeting Briefing

**As a** user, **I want** a pre-meeting briefing, **so that** I'm prepared before recording.

### Tasks
- [ ] **6.1.1** Add "Start Meeting" flow with client selection
- [ ] **6.1.2** Load client profile, historical notes, vendor overlap
- [ ] **6.1.3** Display briefing summary before recording starts
- [ ] **6.1.4** Allow user to review and start recording

### Test Plans

**Unit Tests:**
- Client data loads correctly
- Briefing summary generated

**Functional Tests:**
- Select client → briefing displays → start recording

**E2E Tests:**
- Select "Delta Airlines" → verify briefing shows prior meetings → start recording → verify LLM has context

### Dependencies & Blockers
- **Dependencies:** Story 3.1, 1.3
- **Blockers:** None

### Acceptance Criteria
- ✅ Client selection triggers briefing
- ✅ Briefing shows profile + history
- ✅ User can start recording from briefing

---

# EPIC 7: Post-Meeting Artifacts

**Goal:** Generate transcript, summary, action items after meeting  
**Dependencies:** Story 2.2 (transcript), Story 4.1 (LLM), Story 1.4 (file storage)

---

## Story 7.1: Artifact Generation

**As a** user, **I want** post-meeting artifacts, **so that** I have deliverables to share.

### Tasks
- [ ] **7.1.1** Trigger artifact generation on meeting end
- [ ] **7.1.2** Generate full transcript markdown
- [ ] **7.1.3** Use LLM to generate meeting summary
- [ ] **7.1.4** Extract action items from transcript
- [ ] **7.1.5** Suggest client record updates (human-in-the-loop)
- [ ] **7.1.6** Save artifacts to flat files

### Test Plans

**Unit Tests:**
- Transcript export works
- Summary generation works
- Action items extracted

**Functional Tests:**
- End meeting → artifacts generated → files saved

**E2E Tests:**
- Complete meeting → verify transcript, summary, action-items files created

### Dependencies & Blockers
- **Dependencies:** Story 2.2, 4.1, 1.4
- **Blockers:** None

### Acceptance Criteria
- ✅ Transcript saved as markdown
- ✅ Summary generated
- ✅ Action items extracted
- ✅ Client update suggestions provided

---

# EPIC 8: Settings & Configuration

**Goal:** Configure STT, LLM, storage, preferences  
**Dependencies:** All prior epics

---

## Story 8.1: Settings UI

**As a** user, **I want** a settings panel, **so that** I can configure endpoints.

### Tasks
- [ ] **8.1.1** Create settings UI
- [ ] **8.1.2** Add fields: STT endpoint, LLM endpoint, API keys, storage path
- [ ] **8.1.3** Validate endpoint URLs
- [ ] **8.1.4** Test connection before saving
- [ ] **8.1.5** Store settings in JSON config file

### Test Plans

**Unit Tests:**
- Settings save/load correctly
- Validation works

**Functional Tests:**
- Update STT endpoint → test connection → save → verify persisted

**E2E Tests:**
- Open settings → change LLM model → verify used in next meeting

### Dependencies & Blockers
- **Dependencies:** All prior
- **Blockers:** None

### Acceptance Criteria
- ✅ Settings UI
- ✅ Endpoint validation
- ✅ Settings persist

---

# EPIC 9: Packaging & Distribution

**Goal:** Build distributable app for macOS  
**Dependencies:** All prior epics

---

## Story 9.1: macOS App Bundle

**As a** user, **I want** a .dmg installer, **so that** I can install the app easily.

### Tasks
- [ ] **9.1.1** Configure electron-builder for macOS
- [ ] **9.1.2** Code sign app (Apple Developer ID)
- [ ] **9.1.3** Notarize app (Apple notarization service)
- [ ] **9.1.4** Create DMG installer
- [ ] **9.1.5** Test installation on clean macOS system

### Test Plans

**E2E Tests:**
- Download DMG → install → launch → verify app works

### Dependencies & Blockers
- **Dependencies:** All prior
- **Blockers:** Apple Developer account required

### Acceptance Criteria
- ✅ DMG installer created
- ✅ App code-signed and notarized
- ✅ Installs cleanly on macOS

---

## Summary

- **9 Epics** covering foundation → deployment
- **30+ Stories** with full task breakdowns
- **Unit, Functional, Integration, E2E tests** defined for each
- **Dependency graph** shows build order
- **Acceptance criteria** for every story

This plan provides a complete roadmap from empty repo to shippable V1.
