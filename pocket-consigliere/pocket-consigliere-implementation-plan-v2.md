# Pocket Consigliere — V1 Implementation Plan (Hybrid Architecture)

**Version:** 0.2  
**Date:** 2026-02-15  
**Architecture:** Electron + Rust audio addon (napi-rs)  
**LLM Strategy:** Ollama (local) + Cloud APIs (configurable)  
**Status:** Planning  

---

## Overview

This document breaks down the Pocket Consigliere V1 epics and stories (hybrid architecture) into concrete tasks, subtasks, test plans, and dependency maps. Every story includes:
- **Tasks & Subtasks** — granular work items
- **Test Plans** — unit, functional, integration, and E2E tests
- **Dependencies & Blockers** — what must be done first
- **Acceptance Criteria** — how we know it's done
- **AI Assistance Notes** — where AI can help (code generation, debugging)

---

## Build Order (Dependency Map)

```
EPIC 0: Week 1 POC (CRITICAL) ──────────────────┐
    ↓                                          │
EPIC 1: Foundation ──────────────────────────┐ │
    ↓                                        │ │
EPIC 2: Audio Capture (Rust Addon) ─────┐   │ │
    ↓                                    │   │ │
EPIC 3: Speech-to-Text                   │   │ │
    ↓                                    │   │ │
EPIC 4: Knowledge Base                   │   │ │
    ↓                                    │   │ │
EPIC 5: LLM Analysis Engine ←────────────┤   │ │
    ↓                                        │ │
EPIC 6: Live Meeting UI ←───────────────────┘ │
    ↓                                          │
EPIC 10: Health & Diagnostics ←───────────────┘
    ↓
EPIC 7: Auto-Prep Mode (nice-to-have)
    ↓
EPIC 8: Post-Meeting Artifacts (nice-to-have)
    ↓
EPIC 9: Settings & Config
    ↓
EPIC 11: Packaging & Distribution
```

---

# EPIC 0: Week 1 Proof of Concept (CRITICAL PATH)

**Goal:** Validate the entire technical stack before full implementation

**Why this matters:** One-way door decision. MUST prove audio, STT, LLM, and UI chain works.

**Timeline:** 7 days (Week 1)

**Success Criteria:**
- ✅ Rust audio addon works (mic + system audio on macOS)
- ✅ Whisper.cpp transcribes in <10s (GPU-accelerated)
- ✅ Ollama analyzes in <60s
- ✅ Minimal UI displays results

**If this fails:** Pivot to "import audio file" workflow OR reconsider architecture

---

## Story 0.1: Full-Stack Proof of Concept

**As a** developer, **I want** to validate the entire technical chain in Week 1, **so that** we don't build on unproven assumptions.

### Timeline

| Day | Focus | Deliverable |
|-----|-------|-------------|
| 1-2 | Fork Meetily audio → napi-rs addon | `.node` binary compiles, exports API |
| 3 | Test audio capture on macOS | 60s recording works |
| 4 | Whisper.cpp integration | Transcription <10s |
| 5 | Ollama integration | Analysis <60s with context |
| 6 | Minimal UI | Displays transcript + suggestions |
| 7 | E2E test | 30-min simulated meeting |

### Tasks

- [ ] **0.1.1** Fork Meetily audio module (GitHub: Zackriya-Solutions/meeting-minutes)
- [ ] **0.1.2** Extract audio capture code (Rust, uses `cpal` crate)
- [ ] **0.1.3** Adapt to napi-rs bindings
- [ ] **0.1.4** Compile as `.node` addon (macOS ARM64 + x86_64)
- [ ] **0.1.5** Test audio capture (mic + system audio, 60s recording)
- [ ] **0.1.6** Integrate Whisper.cpp (Metal-accelerated)
- [ ] **0.1.7** Test transcription latency (<10s for 60s audio)
- [ ] **0.1.8** Install Ollama, download Llama 3.1 8B model
- [ ] **0.1.9** Build OpenAI-compatible Ollama client
- [ ] **0.1.10** Test LLM analysis (transcript + client context → suggestions in <60s)
- [ ] **0.1.11** Build minimal Electron UI (transcript left, suggestions right)
- [ ] **0.1.12** E2E test: 30-min simulated meeting (audio → STT → LLM → UI)

### Subtasks (0.1.1-0.1.4 — Rust Audio Addon)

**0.1.1: Fork Meetily audio module**
- Clone repo: `git clone https://github.com/Zackriya-Solutions/meeting-minutes meetily`
- Navigate to audio module: `meetily/src-tauri/src/audio/`
- Review code: `capture.rs`, `mixer.rs`, `device_manager.rs`
- Understand dependencies: `cpal`, `hound`, `dasp`

**0.1.2: Extract audio capture code**
- Copy relevant files to new directory: `pocket-consigliere/src/rust-addon/audio/`
- Remove Tauri-specific code (keep core audio logic)
- Preserve: mic + system audio mixing, ducking, clipping prevention

**0.1.3: Adapt to napi-rs bindings**
- Initialize napi-rs project:
  ```bash
  npm install -g @napi-rs/cli
  napi new pocket-consigliere-audio
  ```
- Define API in `lib.rs`:
  ```rust
  #[napi]
  pub struct AudioCapture { /* ... */ }
  
  #[napi]
  impl AudioCapture {
    #[napi(constructor)]
    pub fn new() -> Self { /* ... */ }
    
    #[napi]
    pub fn start_capture(&self) -> Result<()> { /* ... */ }
    
    #[napi]
    pub fn stop_capture(&self) -> Result<()> { /* ... */ }
    
    #[napi]
    pub fn get_buffer(&self) -> Buffer { /* ... */ }
    
    #[napi]
    pub fn get_status(&self) -> String { /* ... */ }
  }
  ```
- Use AI: "Convert this Rust audio code to napi-rs bindings with these methods"

**0.1.4: Compile as `.node` addon**
- Build for macOS:
  ```bash
  cargo build --release
  # Output: target/release/libpocket_consigliere_audio.dylib
  # napi-rs renames to: audio.darwin-arm64.node
  ```
- Test in Node.js:
  ```javascript
  const audio = require('./audio.darwin-arm64.node');
  const capture = new audio.AudioCapture();
  console.log(capture.getStatus()); // "Ready"
  ```

### Subtasks (0.1.5 — Test Audio Capture)

- Install BlackHole (macOS virtual audio driver):
  ```bash
  brew install blackhole-2ch
  ```
- Configure macOS audio:
  - System Preferences → Sound → Output → BlackHole 2ch
  - Create Multi-Output Device (BlackHole + Built-in Output)
- Test script:
  ```javascript
  const audio = require('./audio.node');
  const capture = new audio.AudioCapture();
  
  capture.startCapture();
  setTimeout(() => {
    const buffer = capture.getBuffer(); // 60s of audio
    capture.stopCapture();
    console.log(`Captured ${buffer.length} bytes`);
  }, 60000);
  ```
- Verify: 60s recording saved as `.wav` file, playback confirms mic + system audio

### Subtasks (0.1.6-0.1.7 — Whisper Integration)

**0.1.6: Integrate Whisper.cpp**
- Clone Whisper.cpp:
  ```bash
  git clone https://github.com/ggerganov/whisper.cpp
  cd whisper.cpp
  ```
- Build with Metal support (macOS GPU acceleration):
  ```bash
  make clean
  WHISPER_METAL=1 make
  ```
- Download model:
  ```bash
  bash ./models/download-ggml-model.sh base.en
  # Output: models/ggml-base.en.bin
  ```
- Test CLI:
  ```bash
  ./main -m models/ggml-base.en.bin -f test-audio.wav
  ```

**0.1.7: Test transcription latency**
- Create Node.js wrapper:
  ```javascript
  const { execSync } = require('child_process');
  
  function transcribe(audioPath) {
    const start = Date.now();
    const output = execSync(
      `./whisper.cpp/main -m models/ggml-base.en.bin -f ${audioPath}`,
      { encoding: 'utf-8' }
    );
    const elapsed = Date.now() - start;
    return { transcript: output, latency: elapsed };
  }
  ```
- Test with 60s audio file:
  ```javascript
  const result = transcribe('test-60s.wav');
  console.log(`Transcription took ${result.latency}ms`);
  // Target: <10,000ms (10s)
  ```

### Subtasks (0.1.8-0.1.10 — Ollama Integration)

**0.1.8: Install Ollama**
- Install via Homebrew:
  ```bash
  brew install ollama
  ollama serve  # Starts server on localhost:11434
  ```
- Download model:
  ```bash
  ollama pull llama3.1:8b
  # Or: ollama pull mixtral:8x7b (larger, slower, better quality)
  ```

**0.1.9: Build Ollama client**
- Ollama exposes OpenAI-compatible API at `http://localhost:11434/v1/`
- Node.js client:
  ```javascript
  const axios = require('axios');
  
  async function generateSuggestions(transcript, clientContext) {
    const response = await axios.post('http://localhost:11434/v1/chat/completions', {
      model: 'llama3.1:8b',
      messages: [
        {
          role: 'system',
          content: `You are a consultant's AI copilot. Generate strategic suggestions based on meeting transcript and client context.`
        },
        {
          role: 'user',
          content: `Transcript:\n${transcript}\n\nClient:\n${clientContext}\n\nGenerate 3 suggestions (questions, notes, or research topics).`
        }
      ],
      max_tokens: 500,
      temperature: 0.7
    });
    return response.data.choices[0].message.content;
  }
  ```

**0.1.10: Test LLM latency**
- Test with sample data:
  ```javascript
  const transcript = "Client mentioned they're using Kubernetes on-prem and struggling with network policies.";
  const clientContext = "TechCorp, fintech, tech stack: Kubernetes, PostgreSQL, Redis";
  
  const start = Date.now();
  const suggestions = await generateSuggestions(transcript, clientContext);
  const elapsed = Date.now() - start;
  
  console.log(`Suggestions generated in ${elapsed}ms`);
  console.log(suggestions);
  // Target: <60,000ms (60s), acceptable <90s
  ```

### Subtasks (0.1.11 — Minimal UI)

- Create Electron app:
  ```bash
  npm init -y
  npm install electron vite react react-dom
  ```
- Minimal UI (split-pane):
  ```jsx
  // App.jsx
  function App() {
    const [transcript, setTranscript] = useState('');
    const [suggestions, setSuggestions] = useState([]);
    
    return (
      <div style={{ display: 'flex', height: '100vh' }}>
        <div style={{ flex: 1, padding: '1rem', overflowY: 'scroll' }}>
          <h2>Transcript</h2>
          <p>{transcript}</p>
        </div>
        <div style={{ flex: 1, padding: '1rem', overflowY: 'scroll' }}>
          <h2>Suggestions</h2>
          <ul>
            {suggestions.map((s, i) => <li key={i}>{s}</li>)}
          </ul>
        </div>
      </div>
    );
  }
  ```

### Subtasks (0.1.12 — E2E Test)

- Simulate 30-min meeting:
  1. Start audio capture
  2. Play pre-recorded meeting audio (or have conversation)
  3. Every 60s:
     - Get audio buffer from Rust addon
     - Transcribe with Whisper
     - Send transcript to Ollama
     - Display suggestions in UI
  4. After 30 min, stop capture
  5. Review: Did suggestions make sense? Was latency acceptable?

### Test Plans

**Unit Tests:**
- Rust addon compiles for macOS ARM64 + x86_64
- `AudioCapture.start()` / `.stop()` / `.getBuffer()` work
- Whisper CLI returns valid transcript
- Ollama API returns valid JSON response

**Functional Tests:**
- Audio capture: 60s recording contains mic + system audio
- Whisper: Transcription accuracy >95% (manual review)
- Ollama: Suggestions are relevant to transcript + context

**Integration Tests:**
- Audio → STT pipeline: 60s audio → transcript in <10s
- STT → LLM pipeline: transcript → suggestions in <60s
- Full chain: audio → STT → LLM → UI displays results

**E2E Tests:**
- 30-min simulated meeting completes without crashes
- UI updates every 60-90s with new suggestions
- Latency stays within acceptable range throughout meeting

### Dependencies & Blockers

- **Dependencies:** None (this is the starting point)
- **Blockers:** If any step fails, STOP and reassess architecture

### Acceptance Criteria

- ✅ Rust audio addon compiles and captures audio on macOS
- ✅ Whisper transcribes 60s audio in <10s (GPU-accelerated)
- ✅ Ollama generates suggestions in <60s
- ✅ Minimal UI displays transcript + suggestions
- ✅ 30-min E2E test completes successfully

### AI Assistance Notes

**Where AI helps:**
- **Rust → napi-rs bindings:** "Convert this Rust audio code to napi-rs with TypeScript types"
- **Whisper wrapper:** "Write a Node.js wrapper for Whisper.cpp CLI"
- **Ollama client:** "Generate OpenAI-compatible client for Ollama API"
- **Debugging Rust errors:** Paste compiler error → AI explains + suggests fix

### Deliverable

**Week 1 POC artifacts:**
- `src/rust-addon/audio.node` (compiled Rust addon)
- `scripts/poc-audio.js` (audio capture test)
- `scripts/poc-whisper.js` (STT test)
- `scripts/poc-ollama.js` (LLM test)
- `src/renderer/poc-ui.jsx` (minimal UI)
- `scripts/poc-e2e.js` (30-min E2E test)

**Go/No-Go Decision:** If POC succeeds, proceed to Epic 1. If it fails, pivot to "import audio file" workflow.

---

# EPIC 1: Foundation — App Shell & Data Layer

**Goal:** Build production-ready Electron app with databases, file storage, IPC

**Timeline:** Week 2

**Dependencies:** Epic 0 (Week 1 POC validated)

---

## Story 1.1: Electron App Scaffold

**As a** developer, **I want** a production-ready Electron app with React frontend, **so that** all features have a runtime.

### Tasks

- [ ] **1.1.1** Initialize Electron project with electron-builder
- [ ] **1.1.2** Configure TypeScript for main + renderer processes
- [ ] **1.1.3** Set up React + Vite in renderer (hot-reload)
- [ ] **1.1.4** Create typed IPC bridge (preload.ts + contextBridge)
- [ ] **1.1.5** Implement clean app lifecycle (no orphan processes)
- [ ] **1.1.6** Set up project structure: `src/main/`, `src/renderer/`, `src/shared/`, `src/rust-addon/`

### Subtasks (1.1.1 — Initialize Project)

- Install dependencies:
  ```bash
  npm init -y
  npm install electron electron-builder
  npm install -D typescript @types/node @types/react
  ```
- Configure `package.json`:
  ```json
  {
    "name": "pocket-consigliere",
    "main": "dist/main/index.js",
    "scripts": {
      "dev": "electron .",
      "build": "tsc && electron-builder",
      "package": "electron-builder --mac"
    }
  }
  ```

### Subtasks (1.1.2 — TypeScript Config)

- `tsconfig.json` (main process):
  ```json
  {
    "compilerOptions": {
      "target": "ES2020",
      "module": "commonjs",
      "outDir": "./dist/main",
      "esModuleInterop": true,
      "strict": true
    },
    "include": ["src/main/**/*"]
  }
  ```
- `tsconfig.renderer.json`:
  ```json
  {
    "compilerOptions": {
      "target": "ES2020",
      "module": "esnext",
      "jsx": "react-jsx",
      "outDir": "./dist/renderer",
      "esModuleInterop": true,
      "strict": true
    },
    "include": ["src/renderer/**/*"]
  }
  ```

### Subtasks (1.1.3 — Vite + React)

- Install Vite:
  ```bash
  npm install -D vite @vitejs/plugin-react
  ```
- `vite.config.ts`:
  ```typescript
  import { defineConfig } from 'vite';
  import react from '@vitejs/plugin-react';
  
  export default defineConfig({
    plugins: [react()],
    root: 'src/renderer',
    build: {
      outDir: '../../dist/renderer'
    }
  });
  ```

### Subtasks (1.1.4 — Typed IPC Bridge)

- `src/shared/ipc-types.ts`:
  ```typescript
  export interface IPCChannels {
    'db:query': { request: string; response: any[] };
    'audio:start': { request: void; response: boolean };
    'audio:stop': { request: void; response: void };
    // ... more channels
  }
  ```
- `src/main/preload.ts`:
  ```typescript
  import { contextBridge, ipcRenderer } from 'electron';
  import { IPCChannels } from '../shared/ipc-types';
  
  contextBridge.exposeInMainWorld('api', {
    invoke: <K extends keyof IPCChannels>(
      channel: K,
      data: IPCChannels[K]['request']
    ): Promise<IPCChannels[K]['response']> => {
      return ipcRenderer.invoke(channel, data);
    }
  });
  ```

### Test Plans

**Unit Tests:**
- Main process launches without error
- Renderer process loads React app
- IPC channels are typed correctly (TypeScript enforces)

**Functional Tests:**
- App window opens on launch
- Hot-reload triggers on file change (Vite)
- App quits cleanly without orphan processes

**Integration Tests:**
- Main ↔ Renderer IPC communication works
- Multiple IPC calls in parallel

**E2E Tests:**
- Launch app → verify window visible → send test IPC call → verify response → quit app

### Dependencies & Blockers

- **Dependencies:** Story 0.1 (Week 1 POC provides foundation)
- **Blockers:** None

### Acceptance Criteria

- ✅ Electron app launches with React UI on macOS
- ✅ Hot-reload works (Vite)
- ✅ Typed IPC bridge enforces contracts

---

## Story 1.2: SQLite Database Layer

**As a** developer, **I want** embedded SQLite auto-initializing on launch, **so that** structured data persists.

### Tasks

- [ ] **1.2.1** Install better-sqlite3 (faster than node-sqlite3)
- [ ] **1.2.2** Define database schema (SQL migrations)
- [ ] **1.2.3** Implement auto-migration on app launch
- [ ] **1.2.4** Create CRUD operations for each table
- [ ] **1.2.5** Expose database operations via IPC

### Subtasks (1.2.1 — Install SQLite)

- Install dependency:
  ```bash
  npm install better-sqlite3
  npm install -D @types/better-sqlite3
  ```

### Subtasks (1.2.2 — Define Schema)

- `src/main/db/schema.sql`:
  ```sql
  -- Clients table
  CREATE TABLE IF NOT EXISTS clients (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    industry TEXT,
    tech_stack_summary TEXT,
    notes TEXT,
    logo_path TEXT,
    created_at TEXT DEFAULT CURRENT_TIMESTAMP,
    updated_at TEXT DEFAULT CURRENT_TIMESTAMP
  );
  
  -- Vendors table
  CREATE TABLE IF NOT EXISTS vendors (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    category TEXT,
    description TEXT,
    website TEXT,
    created_at TEXT DEFAULT CURRENT_TIMESTAMP
  );
  
  -- Products table
  CREATE TABLE IF NOT EXISTS products (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    vendor_id INTEGER NOT NULL,
    name TEXT NOT NULL,
    description TEXT,
    category TEXT,
    FOREIGN KEY (vendor_id) REFERENCES vendors(id) ON DELETE CASCADE
  );
  
  -- Meetings table
  CREATE TABLE IF NOT EXISTS meetings (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    client_id INTEGER NOT NULL,
    title TEXT,
    meeting_type TEXT, -- discovery, demo, technical, etc.
    start_time TEXT,
    end_time TEXT,
    transcript_path TEXT,
    recording_path TEXT,
    summary TEXT,
    created_at TEXT DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (client_id) REFERENCES clients(id) ON DELETE CASCADE
  );
  
  -- Transcript chunks table
  CREATE TABLE IF NOT EXISTS transcript_chunks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    meeting_id INTEGER NOT NULL,
    chunk_index INTEGER NOT NULL,
    text TEXT NOT NULL,
    timestamp TEXT,
    FOREIGN KEY (meeting_id) REFERENCES meetings(id) ON DELETE CASCADE
  );
  
  -- Suggestions table
  CREATE TABLE IF NOT EXISTS suggestions (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    meeting_id INTEGER NOT NULL,
    type TEXT NOT NULL, -- question, note, research, action
    text TEXT NOT NULL,
    status TEXT DEFAULT 'pending', -- pending, acknowledged, dismissed
    timestamp TEXT DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (meeting_id) REFERENCES meetings(id) ON DELETE CASCADE
  );
  
  -- Client tech stack history (for tracking changes over time)
  CREATE TABLE IF NOT EXISTS client_stack_history (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    client_id INTEGER NOT NULL,
    snapshot_date TEXT DEFAULT CURRENT_TIMESTAMP,
    tech_stack TEXT,
    FOREIGN KEY (client_id) REFERENCES clients(id) ON DELETE CASCADE
  );
  ```

### Subtasks (1.2.3 — Auto-Migration)

- `src/main/db/index.ts`:
  ```typescript
  import Database from 'better-sqlite3';
  import { readFileSync } from 'fs';
  import { join } from 'path';
  import { app } from 'electron';
  
  const dbPath = join(app.getPath('userData'), 'pocket-consigliere.db');
  const db = new Database(dbPath);
  
  // Run schema on startup
  const schema = readFileSync(join(__dirname, 'schema.sql'), 'utf-8');
  db.exec(schema);
  
  export default db;
  ```

### Subtasks (1.2.4 — CRUD Operations)

- `src/main/db/clients.ts`:
  ```typescript
  import db from './index';
  
  export interface Client {
    id?: number;
    name: string;
    industry?: string;
    tech_stack_summary?: string;
    notes?: string;
    logo_path?: string;
  }
  
  export const createClient = (client: Client) => {
    const stmt = db.prepare(`
      INSERT INTO clients (name, industry, tech_stack_summary, notes, logo_path)
      VALUES (?, ?, ?, ?, ?)
    `);
    const info = stmt.run(
      client.name,
      client.industry || null,
      client.tech_stack_summary || null,
      client.notes || null,
      client.logo_path || null
    );
    return info.lastInsertRowid;
  };
  
  export const getClient = (id: number): Client | null => {
    const stmt = db.prepare('SELECT * FROM clients WHERE id = ?');
    return stmt.get(id) as Client | null;
  };
  
  export const updateClient = (id: number, client: Partial<Client>) => {
    const fields = Object.keys(client).map(k => `${k} = ?`).join(', ');
    const values = Object.values(client);
    const stmt = db.prepare(`UPDATE clients SET ${fields}, updated_at = CURRENT_TIMESTAMP WHERE id = ?`);
    stmt.run(...values, id);
  };
  
  export const deleteClient = (id: number) => {
    const stmt = db.prepare('DELETE FROM clients WHERE id = ?');
    stmt.run(id);
  };
  
  export const listClients = (): Client[] => {
    const stmt = db.prepare('SELECT * FROM clients ORDER BY name');
    return stmt.all() as Client[];
  };
  ```

### Subtasks (1.2.5 — Expose via IPC)

- `src/main/ipc/db.ts`:
  ```typescript
  import { ipcMain } from 'electron';
  import * as clientsDB from '../db/clients';
  import * as vendorsDB from '../db/vendors';
  // ... import other DB modules
  
  ipcMain.handle('db:clients:create', (event, client) => {
    return clientsDB.createClient(client);
  });
  
  ipcMain.handle('db:clients:get', (event, id) => {
    return clientsDB.getClient(id);
  });
  
  // ... expose all CRUD operations
  ```

### Test Plans

**Unit Tests:**
- Database file created at correct location
- Schema migrations run successfully
- CRUD operations work for each table

**Functional Tests:**
- Create client → retrieve → update → delete (full cycle)
- Foreign key constraints enforced (deleting client cascades to meetings)
- Data persists across app restarts

**Integration Tests:**
- IPC calls trigger database operations
- Multiple concurrent database operations don't corrupt data

**E2E Tests:**
- Launch app → create client via UI → restart app → client still exists

### Dependencies & Blockers

- **Dependencies:** Story 1.1 (Electron app scaffold)
- **Blockers:** None

### Acceptance Criteria

- ✅ Database initializes automatically on launch
- ✅ All tables created with correct schema
- ✅ CRUD operations work for all entities
- ✅ Data persists across restarts

---

## Story 1.3: Vector Database Layer

**As a** developer, **I want** embedded vector database, **so that** semantic search works on clients, vendors, transcripts.

### Tasks

- [ ] **1.3.1** Choose vector DB (LanceDB preferred, ChromaDB alternative)
- [ ] **1.3.2** Install embedding model (all-MiniLM-L6-v2, local)
- [ ] **1.3.3** Initialize vector DB on app launch
- [ ] **1.3.4** Implement insert/search operations
- [ ] **1.3.5** Integrate with SQLite (vector IDs → foreign keys)

### Subtasks (1.3.1 — Choose Vector DB)

**Option A: LanceDB (recommended)**
- Pure Rust, embedded, no server
- Faster than ChromaDB for local use
- Better TypeScript support

**Option B: ChromaDB**
- Python-based, requires local server
- More mature ecosystem
- Slightly slower for embedded use

**Decision:** Use LanceDB for v1 (simpler embedding)

### Subtasks (1.3.2 — Install Embedding Model)

- Install dependencies:
  ```bash
  npm install @xenova/transformers
  ```
- Load model:
  ```typescript
  import { pipeline } from '@xenova/transformers';
  
  const embedder = await pipeline('feature-extraction', 'Xenova/all-MiniLM-L6-v2');
  
  async function embed(text: string): Promise<number[]> {
    const output = await embedder(text, { pooling: 'mean', normalize: true });
    return Array.from(output.data);
  }
  ```

### Subtasks (1.3.3 — Initialize LanceDB)

- Install:
  ```bash
  npm install vectordb
  ```
- Initialize:
  ```typescript
  import { connect } from 'vectordb';
  import { join } from 'path';
  import { app } from 'electron';
  
  const dbPath = join(app.getPath('userData'), 'vector-db');
  const vectorDB = await connect(dbPath);
  
  // Create tables
  await vectorDB.createTable('clients', [
    { id: 1, embedding: [/* 384 dimensions */], text: "Sample client" }
  ]);
  
  await vectorDB.createTable('transcripts', [
    { id: 1, embedding: [/* 384 dimensions */], text: "Sample transcript" }
  ]);
  ```

### Subtasks (1.3.4 — Insert/Search)

- Insert document:
  ```typescript
  async function insertClient(id: number, text: string) {
    const embedding = await embed(text);
    const table = await vectorDB.openTable('clients');
    await table.add([{ id, embedding, text }]);
  }
  ```
- Semantic search:
  ```typescript
  async function searchClients(query: string, topK: number = 5) {
    const queryEmbedding = await embed(query);
    const table = await vectorDB.openTable('clients');
    const results = await table.search(queryEmbedding).limit(topK).execute();
    return results.map(r => ({ id: r.id, text: r.text, score: r.score }));
  }
  ```

### Test Plans

**Unit Tests:**
- Vector DB initializes successfully
- Embeddings generated correctly (384 dimensions)
- Insert operation adds vectors

**Functional Tests:**
- Semantic search returns relevant results
- Query: "kubernetes orchestration" → finds clients using K8s
- Performance: search completes in <100ms

**Integration Tests:**
- Vector DB IDs match SQLite IDs
- Deleting client removes vector entry

**E2E Tests:**
- Add client → embed profile → search for related concept → client appears in results

### Dependencies & Blockers

- **Dependencies:** Story 1.1 (Electron app)
- **Blockers:** None

### Acceptance Criteria

- ✅ Vector DB initializes on launch
- ✅ Documents can be inserted with embeddings
- ✅ Semantic search returns relevant results (<100ms)
- ✅ Integration with SQLite (foreign keys)

---

## Story 1.4: Flat File Storage System

**As a** developer, **I want** structured file storage, **so that** large files don't bloat the database.

### Tasks

- [ ] **1.4.1** Define folder structure
- [ ] **1.4.2** Implement file save/load operations
- [ ] **1.4.3** Store file paths in SQLite (not contents)
- [ ] **1.4.4** Implement orphaned file cleanup

### Subtasks (1.4.1 — Folder Structure)

- Create directories on app launch:
  ```typescript
  import { mkdir } from 'fs/promises';
  import { join } from 'path';
  import { app } from 'electron';
  
  const appDataPath = app.getPath('userData');
  const dirs = ['transcripts', 'recordings', 'briefings'];
  
  for (const dir of dirs) {
    await mkdir(join(appDataPath, dir), { recursive: true });
  }
  ```

### Subtasks (1.4.2 — Save/Load Operations)

- Save file:
  ```typescript
  import { writeFile } from 'fs/promises';
  
  async function saveTranscript(meetingId: number, content: string) {
    const filePath = join(appDataPath, 'transcripts', `${meetingId}.txt`);
    await writeFile(filePath, content, 'utf-8');
    return filePath;
  }
  ```
- Load file:
  ```typescript
  import { readFile } from 'fs/promises';
  
  async function loadTranscript(meetingId: number): Promise<string> {
    const filePath = join(appDataPath, 'transcripts', `${meetingId}.txt`);
    return await readFile(filePath, 'utf-8');
  }
  ```

### Subtasks (1.4.3 — Store Paths in SQLite)

- When saving meeting:
  ```typescript
  const transcriptPath = await saveTranscript(meetingId, transcript);
  
  const stmt = db.prepare(`
    UPDATE meetings 
    SET transcript_path = ? 
    WHERE id = ?
  `);
  stmt.run(transcriptPath, meetingId);
  ```

### Subtasks (1.4.4 — Orphaned File Cleanup)

- On app startup:
  ```typescript
  import { readdir, unlink } from 'fs/promises';
  
  async function cleanupOrphanedFiles() {
    const transcripts = await readdir(join(appDataPath, 'transcripts'));
    
    for (const file of transcripts) {
      const meetingId = parseInt(file.replace('.txt', ''));
      const meeting = db.prepare('SELECT id FROM meetings WHERE id = ?').get(meetingId);
      
      if (!meeting) {
        await unlink(join(appDataPath, 'transcripts', file));
        console.log(`Deleted orphaned transcript: ${file}`);
      }
    }
  }
  ```

### Test Plans

**Unit Tests:**
- Directories created on app launch
- Save/load operations work

**Functional Tests:**
- File saved → path stored in database → file retrieved correctly
- Deleting meeting removes file (or leaves orphan for cleanup)

**Integration Tests:**
- Orphaned file cleanup runs on startup
- Large files (10 MB+) don't slow down database queries

**E2E Tests:**
- Create meeting → save transcript → restart app → load transcript → verify content

### Dependencies & Blockers

- **Dependencies:** Story 1.2 (SQLite database)
- **Blockers:** None

### Acceptance Criteria

- ✅ Files stored in structured directories
- ✅ File paths stored in SQLite (not contents)
- ✅ Orphaned files cleaned up automatically

---

## Story 1.5: IPC Communication Layer

**As a** developer, **I want** typed IPC channels, **so that** frontend can trigger backend operations safely.

### Tasks

- [ ] **1.5.1** Define all IPC channels in shared types file
- [ ] **1.5.2** Implement typed preload script
- [ ] **1.5.3** Create IPC handlers for all database operations
- [ ] **1.5.4** Add error handling (Result<T, E> pattern)

### Subtasks (1.5.1 — Define IPC Channels)

- `src/shared/ipc-types.ts`:
  ```typescript
  export interface IPCChannels {
    // Database operations
    'db:clients:create': { request: Client; response: number };
    'db:clients:get': { request: number; response: Client | null };
    'db:clients:list': { request: void; response: Client[] };
    'db:clients:update': { request: { id: number; data: Partial<Client> }; response: void };
    'db:clients:delete': { request: number; response: void };
    
    // Audio operations
    'audio:start': { request: void; response: boolean };
    'audio:stop': { request: void; response: void };
    'audio:status': { request: void; response: string };
    
    // STT operations
    'stt:transcribe': { request: string; response: string }; // path → transcript
    
    // LLM operations
    'llm:analyze': { request: { transcript: string; context: any }; response: string[] };
    
    // File operations
    'file:save-transcript': { request: { meetingId: number; content: string }; response: string };
    'file:load-transcript': { request: number; response: string };
  }
  ```

### Subtasks (1.5.2 — Typed Preload)

- Already done in Story 1.1.4, expand here if needed

### Subtasks (1.5.3 — IPC Handlers)

- `src/main/ipc/index.ts`:
  ```typescript
  import { ipcMain } from 'electron';
  import { IPCChannels } from '../shared/ipc-types';
  import * as clientsDB from '../db/clients';
  import * as audio from '../rust-addon/audio.node';
  
  // Database handlers
  ipcMain.handle('db:clients:create', async (event, client: Client) => {
    return clientsDB.createClient(client);
  });
  
  ipcMain.handle('db:clients:get', async (event, id: number) => {
    return clientsDB.getClient(id);
  });
  
  // Audio handlers
  ipcMain.handle('audio:start', async () => {
    try {
      audio.startCapture();
      return true;
    } catch (err) {
      console.error('Audio capture failed:', err);
      return false;
    }
  });
  
  // ... etc for all channels
  ```

### Subtasks (1.5.4 — Error Handling)

- Wrap all IPC handlers in try/catch:
  ```typescript
  ipcMain.handle('db:clients:create', async (event, client) => {
    try {
      return { success: true, data: clientsDB.createClient(client) };
    } catch (err) {
      return { success: false, error: err.message };
    }
  });
  ```
- Frontend handles Result<T, E>:
  ```typescript
  const result = await window.api.invoke('db:clients:create', newClient);
  if (result.success) {
    console.log('Created client:', result.data);
  } else {
    alert(`Error: ${result.error}`);
  }
  ```

### Test Plans

**Unit Tests:**
- TypeScript enforces IPC channel types
- All database operations have IPC handlers

**Functional Tests:**
- Renderer can call all IPC channels
- Errors propagate correctly to frontend

**Integration Tests:**
- Multiple concurrent IPC calls don't block
- IPC calls trigger correct backend operations

**E2E Tests:**
- UI button → IPC call → database update → UI reflects change

### Dependencies & Blockers

- **Dependencies:** Story 1.1 (Electron scaffold), Story 1.2 (SQLite)
- **Blockers:** None

### Acceptance Criteria

- ✅ All IPC channels typed (TypeScript enforces)
- ✅ Error handling propagates to renderer
- ✅ Frontend can trigger all backend operations

---

# EPIC 2: Audio Capture (Rust Addon)

**Goal:** Production-ready audio capture (mic + system audio) using Rust addon

**Timeline:** Week 3

**Dependencies:** Epic 0 (Week 1 POC), Epic 1 (Foundation)

---

## Story 2.1: Fork Meetily Audio Module → napi-rs Addon

**Already completed in Week 1 POC (Story 0.1). This task refines it for production.**

### Tasks

- [ ] **2.1.1** Refactor POC code for production (error handling, logging)
- [ ] **2.1.2** Add TypeScript types (auto-generated by napi-rs)
- [ ] **2.1.3** Compile for both ARM64 + x86_64 (universal binary)
- [ ] **2.1.4** Add unit tests for Rust code
- [ ] **2.1.5** Document API for future maintainers

### Subtasks (2.1.1 — Production Refactor)

- Add structured logging:
  ```rust
  use log::{info, warn, error};
  
  #[napi]
  impl AudioCapture {
    #[napi]
    pub fn start_capture(&self) -> Result<()> {
      info!("Starting audio capture");
      // ... implementation
      match result {
        Ok(_) => {
          info!("Audio capture started successfully");
          Ok(())
        }
        Err(e) => {
          error!("Audio capture failed: {}", e);
          Err(napi::Error::from_reason(e.to_string()))
        }
      }
    }
  }
  ```
- Add error types:
  ```rust
  #[derive(Debug)]
  pub enum AudioError {
    DeviceNotFound,
    PermissionDenied,
    CaptureFailedoverride,
  }
  
  impl std::fmt::Display for AudioError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
      match self {
        AudioError::DeviceNotFound => write!(f, "Audio device not found"),
        AudioError::PermissionDenied => write!(f, "Microphone permission denied"),
        AudioError::CaptureFailed => write!(f, "Audio capture failed"),
      }
    }
  }
  ```

### Subtasks (2.1.2 — TypeScript Types)

- napi-rs auto-generates types:
  ```typescript
  // Generated: index.d.ts
  export class AudioCapture {
    constructor();
    startCapture(): void;
    stopCapture(): void;
    getBuffer(): Buffer;
    getStatus(): string;
  }
  ```

### Subtasks (2.1.3 — Universal Binary)

- Build for both architectures:
  ```bash
  # ARM64 (M1/M2 Macs)
  cargo build --release --target aarch64-apple-darwin
  
  # x86_64 (Intel Macs)
  cargo build --release --target x86_64-apple-darwin
  
  # Create universal binary
  lipo -create \
    target/aarch64-apple-darwin/release/libaudio.dylib \
    target/x86_64-apple-darwin/release/libaudio.dylib \
    -output audio.darwin-universal.node
  ```

### Subtasks (2.1.4 — Unit Tests)

- `src/rust-addon/tests/audio_tests.rs`:
  ```rust
  #[cfg(test)]
  mod tests {
    use super::*;
    
    #[test]
    fn test_audio_capture_init() {
      let capture = AudioCapture::new();
      assert_eq!(capture.get_status(), "Ready");
    }
    
    #[test]
    fn test_start_stop() {
      let capture = AudioCapture::new();
      capture.start_capture().unwrap();
      capture.stop_capture();
    }
  }
  ```

### Test Plans

**Unit Tests:**
- Rust tests pass (`cargo test`)
- Binary compiles for ARM64 + x86_64

**Functional Tests:**
- Audio capture works on M1 Mac (ARM64)
- Audio capture works on Intel Mac (x86_64)

**Integration Tests:**
- Node.js can load `.node` addon
- TypeScript types match Rust API

**E2E Tests:**
- Electron app loads addon without errors

### Dependencies & Blockers

- **Dependencies:** Story 0.1 (Week 1 POC)
- **Blockers:** None

### Acceptance Criteria

- ✅ Production-ready Rust addon (error handling, logging)
- ✅ Universal binary (ARM64 + x86_64)
- ✅ Unit tests pass

---

## Story 2.2: System Audio + Mic Capture (macOS)

**Already partially done in POC. This task adds production features.**

### Tasks

- [ ] **2.2.1** Add audio device selection UI
- [ ] **2.2.2** Implement health check (is audio working?)
- [ ] **2.2.3** Add intelligent ducking (prevent feedback)
- [ ] **2.2.4** Add clipping prevention
- [ ] **2.2.5** Store audio as WAV files (for STT processing)

### Subtasks (2.2.1 — Device Selection UI)

- List available devices:
  ```rust
  #[napi]
  pub fn list_input_devices() -> Vec<String> {
    // Use cpal to enumerate devices
    let host = cpal::default_host();
    host.input_devices()
      .unwrap()
      .map(|d| d.name().unwrap())
      .collect()
  }
  ```
- Frontend UI:
  ```jsx
  function AudioSettings() {
    const [devices, setDevices] = useState([]);
    
    useEffect(() => {
      window.api.invoke('audio:list-devices').then(setDevices);
    }, []);
    
    return (
      <select>
        {devices.map(d => <option key={d}>{d}</option>)}
      </select>
    );
  }
  ```

### Subtasks (2.2.2 — Health Check)

- Test audio capture without starting meeting:
  ```rust
  #[napi]
  pub fn test_audio() -> Result<String> {
    // Record 5 seconds
    // Return "OK" or error message
  }
  ```
- Frontend:
  ```jsx
  <button onClick={async () => {
    const result = await window.api.invoke('audio:test');
    alert(result);
  }}>
    Test Audio
  </button>
  ```

### Subtasks (2.2.3 — Intelligent Ducking)

- Already in Meetily's code, verify it's working:
  - When system audio plays, reduce mic volume temporarily
  - Prevents feedback loop

### Subtasks (2.2.4 — Clipping Prevention)

- Normalize audio levels:
  ```rust
  fn normalize_audio(samples: &mut [f32]) {
    let max = samples.iter().map(|s| s.abs()).max_by(|a, b| a.partial_cmp(b).unwrap()).unwrap();
    if max > 1.0 {
      for sample in samples.iter_mut() {
        *sample /= max;
      }
    }
  }
  ```

### Subtasks (2.2.5 — Save as WAV)

- Use `hound` crate (already in Meetily):
  ```rust
  use hound::{WavWriter, WavSpec};
  
  let spec = WavSpec {
    channels: 2,
    sample_rate: 48000,
    bits_per_sample: 16,
    sample_format: hound::SampleFormat::Int,
  };
  
  let mut writer = WavWriter::create("output.wav", spec)?;
  for sample in audio_buffer {
    writer.write_sample(sample)?;
  }
  writer.finalize()?;
  ```

### Test Plans

**Unit Tests:**
- Device listing returns correct devices
- Health check returns "OK" when audio works

**Functional Tests:**
- User selects device → audio capture uses correct device
- Ducking prevents feedback (test with loopback)
- Clipping prevention keeps levels <1.0

**Integration Tests:**
- Audio saved as WAV → can be loaded into Whisper

**E2E Tests:**
- Start meeting → record 60s → stop → WAV file exists → playback confirms audio

### Dependencies & Blockers

- **Dependencies:** Story 2.1 (Rust addon)
- **Blockers:** BlackHole must be installed by user

### Acceptance Criteria

- ✅ Audio device selection UI works
- ✅ Health check shows audio status
- ✅ Ducking + clipping prevention work
- ✅ Audio saved as WAV files

---

## Story 2.3: Audio Fallback — Manual Import

**As a** user, **I want** to import an audio file if live capture fails, **so that** the app is still useful.

### Tasks

- [ ] **2.3.1** Add "Import Audio File" button
- [ ] **2.3.2** Support .wav, .mp3, .m4a, .opus formats
- [ ] **2.3.3** Convert to WAV if needed (for Whisper)
- [ ] **2.3.4** Process imported audio same as live capture

### Subtasks (2.3.1 — Import UI)

- File picker:
  ```jsx
  <button onClick={async () => {
    const filePath = await window.api.invoke('file:open-dialog', {
      filters: [{ name: 'Audio', extensions: ['wav', 'mp3', 'm4a', 'opus'] }]
    });
    if (filePath) {
      await window.api.invoke('audio:import', filePath);
    }
  }}>
    Import Audio File
  </button>
  ```

### Subtasks (2.3.2-2.3.3 — Format Conversion)

- Use FFmpeg to convert to WAV:
  ```bash
  ffmpeg -i input.mp3 -ar 16000 -ac 1 output.wav
  ```
- Node.js wrapper:
  ```typescript
  import { execSync } from 'child_process';
  
  function convertToWav(inputPath: string, outputPath: string) {
    execSync(`ffmpeg -i "${inputPath}" -ar 16000 -ac 1 "${outputPath}"`);
  }
  ```

### Test Plans

**Unit Tests:**
- File picker opens
- Format conversion works (.mp3 → .wav)

**Functional Tests:**
- Import audio → transcribe → display results

**E2E Tests:**
- Import .mp3 file → Whisper transcribes → LLM analyzes → suggestions appear

### Dependencies & Blockers

- **Dependencies:** Story 2.2 (Audio capture), Story 3.1 (Whisper)
- **Blockers:** FFmpeg must be bundled with app

### Acceptance Criteria

- ✅ Import button works
- ✅ Supports .wav, .mp3, .m4a, .opus
- ✅ Imported audio processed same as live capture

---

# EPIC 3: Speech-to-Text

**Goal:** Fast, accurate transcription using Whisper.cpp (local) or cloud STT

**Timeline:** Week 4

**Dependencies:** Epic 2 (Audio Capture)

---

## Story 3.1: Whisper.cpp Integration (GPU-Accelerated)

**Already partially done in POC. This task adds production features.**

### Tasks

- [ ] **3.1.1** Bundle Whisper.cpp binary with app
- [ ] **3.1.2** Download model on first launch (or bundle with app)
- [ ] **3.1.3** Implement Node.js wrapper with streaming support
- [ ] **3.1.4** Add GPU detection (Metal on macOS, fallback to CPU)
- [ ] **3.1.5** Store transcripts in SQLite + vector DB

### Subtasks (3.1.1 — Bundle Whisper Binary)

- Build Whisper.cpp with Metal support:
  ```bash
  cd whisper.cpp
  WHISPER_METAL=1 make
  # Output: whisper.cpp/main (binary)
  ```
- Copy binary to app resources:
  ```bash
  cp whisper.cpp/main resources/whisper-macos-arm64
  ```
- electron-builder config:
  ```json
  {
    "extraResources": [
      { "from": "resources/whisper-macos-arm64", "to": "whisper" }
    ]
  }
  ```

### Subtasks (3.1.2 — Download Model)

- On first launch:
  ```typescript
  import { existsSync } from 'fs';
  import { join } from 'path';
  import { app } from 'electron';
  import { execSync } from 'child_process';
  
  const modelPath = join(app.getPath('userData'), 'models', 'ggml-base.en.bin');
  
  if (!existsSync(modelPath)) {
    console.log('Downloading Whisper model...');
    execSync('bash ./scripts/download-model.sh base.en', { cwd: 'whisper.cpp' });
  }
  ```

### Subtasks (3.1.3 — Node.js Wrapper)

- Streaming transcription:
  ```typescript
  import { spawn } from 'child_process';
  
  function transcribeStream(audioPath: string, onChunk: (text: string) => void) {
    const whisper = spawn('./resources/whisper', [
      '-m', modelPath,
      '-f', audioPath,
      '-t', '4', // 4 threads
      '-p', '0', // no prompt
      '--print-progress'
    ]);
    
    whisper.stdout.on('data', (data) => {
      const text = data.toString().trim();
      if (text) onChunk(text);
    });
    
    whisper.on('close', (code) => {
      console.log(`Whisper exited with code ${code}`);
    });
  }
  ```

### Subtasks (3.1.4 — GPU Detection)

- Check if Metal is available:
  ```typescript
  import { execSync } from 'child_process';
  
  function hasMetalSupport(): boolean {
    try {
      const output = execSync('system_profiler SPDisplaysDataType', { encoding: 'utf-8' });
      return output.includes('Metal');
    } catch {
      return false;
    }
  }
  
  if (hasMetalSupport()) {
    console.log('Using Metal-accelerated Whisper');
  } else {
    console.warn('Metal not available, using CPU (slower)');
  }
  ```

### Subtasks (3.1.5 — Store Transcripts)

- Save transcript chunks:
  ```typescript
  import db from './db';
  
  function saveTranscriptChunk(meetingId: number, chunkIndex: number, text: string) {
    const stmt = db.prepare(`
      INSERT INTO transcript_chunks (meeting_id, chunk_index, text, timestamp)
      VALUES (?, ?, ?, CURRENT_TIMESTAMP)
    `);
    stmt.run(meetingId, chunkIndex, text);
  }
  ```

### Test Plans

**Unit Tests:**
- Whisper binary runs without error
- Model download works on first launch

**Functional Tests:**
- Transcription accuracy >95% (manual review of test audio)
- GPU acceleration detected on M1 Mac
- CPU fallback works on older Macs

**Performance Tests:**
- 60s audio transcribes in <10s on M1 Mac (GPU)
- 60s audio transcribes in <30s on Intel Mac (CPU)

**Integration Tests:**
- Audio → Whisper → transcript saved to database

**E2E Tests:**
- Start meeting → record 60s → transcribe → transcript appears in UI

### Dependencies & Blockers

- **Dependencies:** Story 2.2 (Audio capture)
- **Blockers:** Whisper.cpp must be compiled for macOS

### Acceptance Criteria

- ✅ Whisper.cpp bundled with app
- ✅ GPU acceleration enabled (Metal on macOS)
- ✅ Transcription <10s for 60s audio (GPU)
- ✅ Transcripts stored in SQLite

---

## Story 3.2: Cloud STT Fallback (OpenAI-Compatible)

**As a** user, **I want** to use cloud STT if Whisper is too slow, **so that** I have flexibility.

### Tasks

- [ ] **3.2.1** Add OpenAI-compatible STT client
- [ ] **3.2.2** Store API key in settings (OS keychain)
- [ ] **3.2.3** Implement automatic fallback (if Whisper fails/slow)
- [ ] **3.2.4** Add cost tracking (minutes used, estimated $)

### Subtasks (3.2.1 — Cloud STT Client)

- OpenAI Whisper API:
  ```typescript
  import axios from 'axios';
  import FormData from 'form-data';
  import { createReadStream } from 'fs';
  
  async function transcribeCloud(audioPath: string, apiKey: string): Promise<string> {
    const form = new FormData();
    form.append('file', createReadStream(audioPath));
    form.append('model', 'whisper-1');
    
    const response = await axios.post('https://api.openai.com/v1/audio/transcriptions', form, {
      headers: {
        ...form.getHeaders(),
        'Authorization': `Bearer ${apiKey}`
      }
    });
    
    return response.data.text;
  }
  ```

### Subtasks (3.2.2 — Store API Key)

- Use electron's safeStorage:
  ```typescript
  import { safeStorage } from 'electron';
  import db from './db';
  
  function saveAPIKey(key: string) {
    const encrypted = safeStorage.encryptString(key);
    const stmt = db.prepare('INSERT OR REPLACE INTO settings (key, value) VALUES (?, ?)');
    stmt.run('stt_api_key', encrypted.toString('base64'));
  }
  
  function getAPIKey(): string {
    const stmt = db.prepare('SELECT value FROM settings WHERE key = ?');
    const row = stmt.get('stt_api_key');
    if (!row) return '';
    
    const encrypted = Buffer.from(row.value, 'base64');
    return safeStorage.decryptString(encrypted);
  }
  ```

### Subtasks (3.2.3 — Automatic Fallback)

- Try Whisper first, fall back to cloud:
  ```typescript
  async function transcribe(audioPath: string): Promise<string> {
    try {
      const start = Date.now();
      const transcript = await transcribeWhisper(audioPath);
      const elapsed = Date.now() - start;
      
      if (elapsed > 30000) { // >30s is too slow
        console.warn('Whisper too slow, will use cloud next time');
      }
      
      return transcript;
    } catch (err) {
      console.error('Whisper failed, falling back to cloud:', err);
      const apiKey = getAPIKey();
      if (!apiKey) throw new Error('No STT API key configured');
      return await transcribeCloud(audioPath, apiKey);
    }
  }
  ```

### Subtasks (3.2.4 — Cost Tracking)

- Track API usage:
  ```typescript
  import db from './db';
  
  function logSTTUsage(durationSeconds: number, cost: number) {
    const stmt = db.prepare(`
      INSERT INTO stt_usage (timestamp, duration_seconds, cost)
      VALUES (CURRENT_TIMESTAMP, ?, ?)
    `);
    stmt.run(durationSeconds, cost);
  }
  
  // OpenAI pricing: $0.006 per minute
  const costPerMinute = 0.006;
  const durationMinutes = audioDuration / 60;
  const cost = durationMinutes * costPerMinute;
  logSTTUsage(audioDuration, cost);
  ```

### Test Plans

**Unit Tests:**
- Cloud STT client works with valid API key
- API key storage/retrieval works

**Functional Tests:**
- Fallback triggers when Whisper fails
- Cost tracking accumulates correctly

**Integration Tests:**
- Whisper fails → cloud STT succeeds

**E2E Tests:**
- Configure cloud STT → disable Whisper → transcribe → cloud STT used

### Dependencies & Blockers

- **Dependencies:** Story 3.1 (Whisper integration)
- **Blockers:** User must provide OpenAI API key

### Acceptance Criteria

- ✅ Cloud STT client works
- ✅ API key stored securely
- ✅ Automatic fallback when Whisper fails
- ✅ Cost tracking implemented

---

# (Continued in next message due to length...)

**Note:** The implementation plan continues with Epics 4-11. Each epic follows the same structure:
- Tasks broken into subtasks
- Test plans (unit, functional, integration, E2E)
- Dependencies & blockers
- Acceptance criteria
- AI assistance notes

**Total estimated timeline:** 7-8 weeks from Week 1 POC to shippable V1.

Would you like me to continue with the remaining epics (4-11)?

---

# EPIC 4: Client & Vendor Knowledge Base

**Goal:** Build CRUD interfaces for clients, vendors, products with semantic search

**Timeline:** Week 4-5

**Dependencies:** Epic 1 (Foundation - SQLite + Vector DB)

---

## Story 4.1: Client Profile Management

**As a** user, **I want** to create/edit client profiles, **so that** the AI knows client context.

### Tasks

- [ ] **4.1.1** Design client profile schema (already done in Epic 1)
- [ ] **4.1.2** Build CRUD UI components (React)
- [ ] **4.1.3** Implement fuzzy search + semantic search
- [ ] **4.1.4** Add client logo upload (optional)
- [ ] **4.1.5** Implement tech stack history tracking

### Subtasks (4.1.2 — CRUD UI)

**Client List View:**
```jsx
// ClientList.jsx
import { useState, useEffect } from 'react';

function ClientList() {
  const [clients, setClients] = useState([]);
  const [searchQuery, setSearchQuery] = useState('');
  
  useEffect(() => {
    loadClients();
  }, []);
  
  async function loadClients() {
    const result = await window.api.invoke('db:clients:list');
    setClients(result);
  }
  
  async function handleSearch(query) {
    setSearchQuery(query);
    if (query.length > 2) {
      // Semantic search
      const results = await window.api.invoke('vector:search:clients', query);
      setClients(results);
    } else {
      loadClients();
    }
  }
  
  return (
    <div>
      <input 
        type="text" 
        placeholder="Search clients..." 
        value={searchQuery}
        onChange={(e) => handleSearch(e.target.value)}
      />
      <button onClick={() => navigate('/clients/new')}>Add Client</button>
      
      <ul>
        {clients.map(client => (
          <li key={client.id}>
            <h3>{client.name}</h3>
            <p>{client.industry}</p>
            <button onClick={() => navigate(`/clients/${client.id}`)}>Edit</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

**Client Detail View:**
```jsx
// ClientDetail.jsx
function ClientDetail({ clientId }) {
  const [client, setClient] = useState(null);
  const [editing, setEditing] = useState(!clientId); // New client = edit mode
  
  useEffect(() => {
    if (clientId) {
      loadClient(clientId);
    }
  }, [clientId]);
  
  async function loadClient(id) {
    const result = await window.api.invoke('db:clients:get', id);
    setClient(result);
  }
  
  async function handleSave() {
    if (clientId) {
      await window.api.invoke('db:clients:update', { id: clientId, data: client });
    } else {
      const newId = await window.api.invoke('db:clients:create', client);
      navigate(`/clients/${newId}`);
    }
    setEditing(false);
  }
  
  return (
    <div>
      {editing ? (
        <form onSubmit={(e) => { e.preventDefault(); handleSave(); }}>
          <input 
            type="text" 
            placeholder="Client Name"
            value={client?.name || ''}
            onChange={(e) => setClient({...client, name: e.target.value})}
          />
          <input 
            type="text" 
            placeholder="Industry"
            value={client?.industry || ''}
            onChange={(e) => setClient({...client, industry: e.target.value})}
          />
          <textarea 
            placeholder="Tech Stack Summary"
            value={client?.tech_stack_summary || ''}
            onChange={(e) => setClient({...client, tech_stack_summary: e.target.value})}
          />
          <textarea 
            placeholder="Notes"
            value={client?.notes || ''}
            onChange={(e) => setClient({...client, notes: e.target.value})}
          />
          <button type="submit">Save</button>
          <button type="button" onClick={() => setEditing(false)}>Cancel</button>
        </form>
      ) : (
        <div>
          <h2>{client?.name}</h2>
          <p><strong>Industry:</strong> {client?.industry}</p>
          <p><strong>Tech Stack:</strong> {client?.tech_stack_summary}</p>
          <p><strong>Notes:</strong> {client?.notes}</p>
          <button onClick={() => setEditing(true)}>Edit</button>
        </div>
      )}
    </div>
  );
}
```

### Subtasks (4.1.3 — Semantic Search)

**Backend (Main Process):**
```typescript
import { embed, searchVectors } from './vector-db';

ipcMain.handle('vector:search:clients', async (event, query: string) => {
  // Embed query
  const queryEmbedding = await embed(query);
  
  // Search vector DB
  const results = await searchVectors('clients', queryEmbedding, 10);
  
  // Fetch full client data from SQLite
  const clientIds = results.map(r => r.id);
  const stmt = db.prepare(`SELECT * FROM clients WHERE id IN (${clientIds.join(',')})`);
  const clients = stmt.all();
  
  // Return with relevance scores
  return clients.map(c => ({
    ...c,
    relevance: results.find(r => r.id === c.id)?.score
  }));
});
```

### Subtasks (4.1.4 — Logo Upload)

```jsx
async function handleLogoUpload(file: File) {
  // Save logo to app data directory
  const logoPath = await window.api.invoke('file:save-logo', {
    clientId: client.id,
    buffer: await file.arrayBuffer(),
    filename: file.name
  });
  
  // Update client with logo path
  await window.api.invoke('db:clients:update', {
    id: client.id,
    data: { logo_path: logoPath }
  });
}
```

**Backend:**
```typescript
ipcMain.handle('file:save-logo', async (event, { clientId, buffer, filename }) => {
  const ext = filename.split('.').pop();
  const logoPath = join(app.getPath('userData'), 'logos', `${clientId}.${ext}`);
  await writeFile(logoPath, Buffer.from(buffer));
  return logoPath;
});
```

### Subtasks (4.1.5 — Tech Stack History)

**On client update, snapshot tech stack:**
```typescript
async function updateClient(id: number, data: Partial<Client>) {
  // Update client
  const stmt = db.prepare(`UPDATE clients SET ... WHERE id = ?`);
  stmt.run(...Object.values(data), id);
  
  // If tech_stack_summary changed, save snapshot
  if (data.tech_stack_summary) {
    const historyStmt = db.prepare(`
      INSERT INTO client_stack_history (client_id, tech_stack)
      VALUES (?, ?)
    `);
    historyStmt.run(id, data.tech_stack_summary);
  }
}
```

**View history:**
```typescript
ipcMain.handle('db:clients:stack-history', async (event, clientId: number) => {
  const stmt = db.prepare(`
    SELECT snapshot_date, tech_stack 
    FROM client_stack_history 
    WHERE client_id = ? 
    ORDER BY snapshot_date DESC
  `);
  return stmt.all(clientId);
});
```

### Test Plans

**Unit Tests:**
- Client CRUD operations work
- Semantic search returns relevant clients

**Functional Tests:**
- Create client → appears in list
- Update client → changes persist
- Delete client → cascades to meetings
- Search "kubernetes" → returns clients using K8s

**Integration Tests:**
- Client saved to SQLite → embedded to vector DB
- Logo upload → file saved → path stored in DB

**E2E Tests:**
- Open app → add client → search for client → edit → save → restart app → client still there

### Dependencies & Blockers

- **Dependencies:** Story 1.2 (SQLite), Story 1.3 (Vector DB), Story 1.5 (IPC)
- **Blockers:** None

### Acceptance Criteria

- ✅ CRUD UI works for clients
- ✅ Fuzzy + semantic search functional
- ✅ Logo upload (optional) works
- ✅ Tech stack history tracked

### AI Assistance Notes

**Where AI helps:**
- "Generate React CRUD components for client profile management"
- "Write semantic search backend for vector DB"
- "Create file upload handler for Electron"

---

## Story 4.2: Vendor & Product Catalog

**As a** user, **I want** a searchable catalog of vendors/products, **so that** AI can suggest solutions.

### Tasks

- [ ] **4.2.1** Design vendor/product schema (already done in Epic 1)
- [ ] **4.2.2** Build CRUD UI for vendors
- [ ] **4.2.3** Build CRUD UI for products (with vendor linkage)
- [ ] **4.2.4** Implement semantic search (e.g., "container orchestration" → K8s vendors)
- [ ] **4.2.5** Add bulk import (CSV upload)

### Subtasks (4.2.2 — Vendor CRUD UI)

```jsx
// VendorList.jsx
function VendorList() {
  const [vendors, setVendors] = useState([]);
  
  useEffect(() => {
    loadVendors();
  }, []);
  
  async function loadVendors() {
    const result = await window.api.invoke('db:vendors:list');
    setVendors(result);
  }
  
  return (
    <div>
      <button onClick={() => navigate('/vendors/new')}>Add Vendor</button>
      <ul>
        {vendors.map(v => (
          <li key={v.id}>
            <h3>{v.name}</h3>
            <p>{v.category}</p>
            <button onClick={() => navigate(`/vendors/${v.id}`)}>Edit</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### Subtasks (4.2.3 — Product CRUD UI)

```jsx
// ProductForm.jsx (inside VendorDetail)
function ProductForm({ vendorId, product, onSave }) {
  const [name, setName] = useState(product?.name || '');
  const [description, setDescription] = useState(product?.description || '');
  const [category, setCategory] = useState(product?.category || '');
  
  async function handleSubmit(e) {
    e.preventDefault();
    const productData = { vendor_id: vendorId, name, description, category };
    
    if (product?.id) {
      await window.api.invoke('db:products:update', { id: product.id, data: productData });
    } else {
      await window.api.invoke('db:products:create', productData);
    }
    
    onSave();
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <input type="text" placeholder="Product Name" value={name} onChange={(e) => setName(e.target.value)} />
      <textarea placeholder="Description" value={description} onChange={(e) => setDescription(e.target.value)} />
      <input type="text" placeholder="Category" value={category} onChange={(e) => setCategory(e.target.value)} />
      <button type="submit">Save Product</button>
    </form>
  );
}
```

### Subtasks (4.2.4 — Semantic Search)

```typescript
ipcMain.handle('vector:search:products', async (event, query: string) => {
  const queryEmbedding = await embed(query);
  const results = await searchVectors('products', queryEmbedding, 10);
  
  // Fetch products with vendor info
  const productIds = results.map(r => r.id);
  const stmt = db.prepare(`
    SELECT p.*, v.name as vendor_name 
    FROM products p 
    JOIN vendors v ON p.vendor_id = v.id 
    WHERE p.id IN (${productIds.join(',')})
  `);
  return stmt.all();
});
```

### Subtasks (4.2.5 — Bulk Import)

```jsx
// CSV format: vendor_name, product_name, category, description
async function handleCSVUpload(file: File) {
  const text = await file.text();
  const rows = text.split('\n').slice(1); // Skip header
  
  for (const row of rows) {
    const [vendorName, productName, category, description] = row.split(',');
    
    // Find or create vendor
    let vendor = await window.api.invoke('db:vendors:find-by-name', vendorName.trim());
    if (!vendor) {
      const vendorId = await window.api.invoke('db:vendors:create', { name: vendorName.trim(), category });
      vendor = { id: vendorId };
    }
    
    // Create product
    await window.api.invoke('db:products:create', {
      vendor_id: vendor.id,
      name: productName.trim(),
      category: category.trim(),
      description: description.trim()
    });
  }
  
  alert('Import complete!');
}
```

### Test Plans

**Unit Tests:**
- Vendor/product CRUD works
- Foreign keys enforced (deleting vendor cascades to products)

**Functional Tests:**
- Create vendor → add products → products linked to vendor
- Search "kubernetes" → returns K8s products
- CSV import → vendors + products created

**Integration Tests:**
- Products embedded to vector DB
- Semantic search across vendors + products

**E2E Tests:**
- Import CSV → search for product → appears in results

### Dependencies & Blockers

- **Dependencies:** Story 1.2 (SQLite), Story 1.3 (Vector DB)
- **Blockers:** None

### Acceptance Criteria

- ✅ CRUD UI for vendors + products
- ✅ Semantic search works
- ✅ CSV bulk import functional

---

# EPIC 5: Real-Time LLM Analysis Engine

**Goal:** Analyze transcripts + context → generate suggestions using Ollama (local) or cloud APIs

**Timeline:** Week 5-6

**Dependencies:** Epic 3 (STT), Epic 4 (Knowledge Base)

---

## Story 5.1: Ollama Integration (Local LLM)

**As a** user, **I want** to use Ollama for local LLM inference, **so that** I don't pay per-token.

### Tasks

- [ ] **5.1.1** Install Ollama + download model (Llama 3.1 8B or Mixtral)
- [ ] **5.1.2** Build OpenAI-compatible client
- [ ] **5.1.3** Implement prompt engineering (system prompt + context injection)
- [ ] **5.1.4** Add timeout handling (>90s = skip cycle)
- [ ] **5.1.5** Optimize for <60s latency (test different models/quantizations)

### Subtasks (5.1.1 — Install Ollama)

**User instructions (in app setup wizard):**
```bash
# Install Ollama
brew install ollama

# Start Ollama server
ollama serve

# Download model
ollama pull llama3.1:8b

# Or larger/better model:
ollama pull mixtral:8x7b
```

### Subtasks (5.1.2 — OpenAI-Compatible Client)

```typescript
// src/main/llm/ollama.ts
import axios from 'axios';

interface Message {
  role: 'system' | 'user' | 'assistant';
  content: string;
}

interface OllamaResponse {
  choices: Array<{
    message: { content: string };
  }>;
}

export async function generateSuggestions(
  transcript: string,
  clientContext: string,
  vendorCatalog: string,
  signal?: AbortSignal
): Promise<string[]> {
  const systemPrompt = `You are a strategic AI copilot for consultants. Analyze meeting transcripts and generate 3 actionable suggestions.

Suggestion types:
- **Questions:** Clarifying questions to ask the client
- **Notes:** Key insights or observations
- **Research:** Topics to investigate after the meeting

Rules:
- Be specific (not generic)
- Avoid redundancy (don't repeat previous suggestions)
- Focus on strategic value (not obvious observations)`;

  const userPrompt = `**Client Context:**
${clientContext}

**Vendor Catalog:**
${vendorCatalog}

**Meeting Transcript (last 60 seconds):**
${transcript}

Generate 3 suggestions (questions, notes, or research topics).`;

  try {
    const response = await axios.post<OllamaResponse>(
      'http://localhost:11434/v1/chat/completions',
      {
        model: 'llama3.1:8b',
        messages: [
          { role: 'system', content: systemPrompt },
          { role: 'user', content: userPrompt }
        ],
        max_tokens: 500,
        temperature: 0.7
      },
      {
        timeout: 90000, // 90s timeout
        signal
      }
    );

    const content = response.data.choices[0].message.content;
    // Parse suggestions (assuming LLM returns numbered list)
    const suggestions = content.split('\n')
      .filter(line => /^\d+\./.test(line))
      .map(line => line.replace(/^\d+\.\s*/, '').trim())
      .slice(0, 3);

    return suggestions;
  } catch (err) {
    if (axios.isCancel(err)) {
      console.warn('LLM request cancelled (timeout)');
    } else {
      console.error('Ollama request failed:', err);
    }
    return [];
  }
}
```

### Subtasks (5.1.3 — Context Injection)

```typescript
// Build client context string
function buildClientContext(clientId: number): string {
  const client = db.prepare('SELECT * FROM clients WHERE id = ?').get(clientId);
  const recentMeetings = db.prepare(`
    SELECT title, start_time, summary 
    FROM meetings 
    WHERE client_id = ? 
    ORDER BY start_time DESC 
    LIMIT 3
  `).all(clientId);

  return `
**Client:** ${client.name}
**Industry:** ${client.industry}
**Tech Stack:** ${client.tech_stack_summary}
**Notes:** ${client.notes}

**Recent Meetings:**
${recentMeetings.map(m => `- ${m.title} (${m.start_time}): ${m.summary}`).join('\n')}
  `.trim();
}

// Build vendor catalog string (semantic search for relevant vendors)
async function buildVendorContext(transcript: string): Promise<string> {
  const relevantProducts = await searchProducts(transcript, 5);
  return relevantProducts
    .map(p => `- ${p.vendor_name}: ${p.name} (${p.description})`)
    .join('\n');
}
```

### Subtasks (5.1.4 — Timeout Handling)

```typescript
ipcMain.handle('llm:analyze', async (event, { transcript, clientId, meetingId }) => {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), 90000); // 90s timeout

  try {
    const clientContext = buildClientContext(clientId);
    const vendorContext = await buildVendorContext(transcript);
    
    const suggestions = await generateSuggestions(
      transcript,
      clientContext,
      vendorContext,
      controller.signal
    );

    clearTimeout(timeoutId);
    return { success: true, suggestions };
  } catch (err) {
    clearTimeout(timeoutId);
    return { success: false, error: err.message };
  }
});
```

### Subtasks (5.1.5 — Latency Optimization)

**Test different models:**
```bash
# Smaller/faster (but lower quality)
ollama pull llama3.1:8b-q4_0  # 4-bit quantization, ~5 GB

# Balanced (recommended)
ollama pull llama3.1:8b  # Default quantization, ~6 GB

# Larger/slower (better quality)
ollama pull mixtral:8x7b  # ~26 GB, 2-3x slower
```

**Benchmark script:**
```typescript
async function benchmarkModels() {
  const testTranscript = "Client mentioned they're struggling with Kubernetes networking...";
  const models = ['llama3.1:8b-q4_0', 'llama3.1:8b', 'mixtral:8x7b'];
  
  for (const model of models) {
    const start = Date.now();
    await generateSuggestions(testTranscript, '', '', model);
    const elapsed = Date.now() - start;
    console.log(`${model}: ${elapsed}ms`);
  }
}
```

### Test Plans

**Unit Tests:**
- Ollama client sends requests correctly
- Timeout triggers after 90s
- Suggestions parsed correctly from LLM output

**Functional Tests:**
- Ollama returns 3 relevant suggestions
- Context injection works (client + vendor info visible in prompt)

**Performance Tests:**
- llama3.1:8b completes in <60s on M1 Mac
- mixtral:8x7b completes in <90s on M2 Max

**Integration Tests:**
- Transcript → Ollama → suggestions saved to database

**E2E Tests:**
- Start meeting → record 60s → Ollama generates suggestions → appear in UI

### Dependencies & Blockers

- **Dependencies:** Story 3.1 (STT), Story 4.1 (Client profiles), Story 4.2 (Vendor catalog)
- **Blockers:** Ollama must be installed by user

### Acceptance Criteria

- ✅ Ollama integration works
- ✅ Suggestions generated in <60s (acceptable <90s)
- ✅ Timeout handling prevents hangs
- ✅ Context injection (client + vendor) works

### AI Assistance Notes

**Where AI helps:**
- "Write OpenAI-compatible client for Ollama API"
- "Generate system prompt for consultant AI copilot"
- "Optimize LLM latency (model selection, quantization)"

---

## Story 5.2: Cloud LLM Integration (OpenAI-Compatible)

**As a** user, **I want** to use cloud LLMs for faster/better suggestions, **so that** I can choose speed vs cost.

### Tasks

- [ ] **5.2.1** Build generic OpenAI-compatible client (reusable for any provider)
- [ ] **5.2.2** Support multiple providers (OpenAI, Anthropic, Groq, OpenRouter)
- [ ] **5.2.3** Store API keys securely (OS keychain)
- [ ] **5.2.4** Add cost tracking (tokens used, estimated $)
- [ ] **5.2.5** Implement automatic failover (cloud → Ollama if API fails)

### Subtasks (5.2.1 — Generic Client)

```typescript
// src/main/llm/cloud.ts
export async function generateSuggestionsCloud(
  transcript: string,
  clientContext: string,
  vendorContext: string,
  config: {
    endpoint: string; // e.g., https://api.openai.com/v1
    apiKey: string;
    model: string; // e.g., gpt-4, claude-3-5-sonnet-20250122
  }
): Promise<string[]> {
  const response = await axios.post(
    `${config.endpoint}/chat/completions`,
    {
      model: config.model,
      messages: [
        { role: 'system', content: SYSTEM_PROMPT },
        { role: 'user', content: buildUserPrompt(transcript, clientContext, vendorContext) }
      ],
      max_tokens: 500,
      temperature: 0.7
    },
    {
      headers: { 'Authorization': `Bearer ${config.apiKey}` },
      timeout: 60000 // 60s timeout (cloud is faster)
    }
  );

  return parseSuggestions(response.data.choices[0].message.content);
}
```

### Subtasks (5.2.2 — Multi-Provider Support)

**Provider configs:**
```typescript
const PROVIDERS = {
  openai: {
    endpoint: 'https://api.openai.com/v1',
    defaultModel: 'gpt-4o'
  },
  anthropic: {
    endpoint: 'https://api.anthropic.com/v1',
    defaultModel: 'claude-3-5-sonnet-20250122',
    headerFormat: 'x-api-key' // Anthropic uses different auth header
  },
  groq: {
    endpoint: 'https://api.groq.com/openai/v1',
    defaultModel: 'llama-3.3-70b-versatile'
  },
  openrouter: {
    endpoint: 'https://openrouter.ai/api/v1',
    defaultModel: 'anthropic/claude-3.5-sonnet'
  }
};
```

### Subtasks (5.2.3 — Secure API Key Storage)

```typescript
import { safeStorage } from 'electron';

function saveAPIKey(provider: string, key: string) {
  const encrypted = safeStorage.encryptString(key);
  db.prepare('INSERT OR REPLACE INTO settings (key, value) VALUES (?, ?)')
    .run(`llm_api_key_${provider}`, encrypted.toString('base64'));
}

function getAPIKey(provider: string): string {
  const row = db.prepare('SELECT value FROM settings WHERE key = ?')
    .get(`llm_api_key_${provider}`);
  if (!row) return '';
  
  const encrypted = Buffer.from(row.value, 'base64');
  return safeStorage.decryptString(encrypted);
}
```

### Subtasks (5.2.4 — Cost Tracking)

```typescript
function calculateCost(provider: string, inputTokens: number, outputTokens: number): number {
  const PRICING = {
    'gpt-4o': { input: 0.00250 / 1000, output: 0.01000 / 1000 }, // per token
    'claude-3-5-sonnet-20250122': { input: 0.00300 / 1000, output: 0.01500 / 1000 },
    'llama-3.3-70b-versatile': { input: 0.00059 / 1000, output: 0.00079 / 1000 }
  };
  
  const pricing = PRICING[provider] || { input: 0, output: 0 };
  return (inputTokens * pricing.input) + (outputTokens * pricing.output);
}

function logLLMUsage(provider: string, model: string, inputTokens: number, outputTokens: number) {
  const cost = calculateCost(model, inputTokens, outputTokens);
  db.prepare(`
    INSERT INTO llm_usage (timestamp, provider, model, input_tokens, output_tokens, cost)
    VALUES (CURRENT_TIMESTAMP, ?, ?, ?, ?, ?)
  `).run(provider, model, inputTokens, outputTokens, cost);
}
```

### Subtasks (5.2.5 — Automatic Failover)

```typescript
async function generateSuggestionsWithFailover(
  transcript: string,
  clientContext: string,
  vendorContext: string
): Promise<string[]> {
  const cloudConfig = getCloudLLMConfig(); // from settings
  
  if (cloudConfig.enabled) {
    try {
      return await generateSuggestionsCloud(transcript, clientContext, vendorContext, cloudConfig);
    } catch (err) {
      console.error('Cloud LLM failed, falling back to Ollama:', err);
    }
  }
  
  // Fallback to Ollama
  return await generateSuggestions(transcript, clientContext, vendorContext);
}
```

### Test Plans

**Unit Tests:**
- API clients work for each provider
- API key storage/retrieval secure
- Cost calculation correct

**Functional Tests:**
- Cloud LLM returns suggestions (<20s)
- Failover triggers when cloud API fails

**Integration Tests:**
- Cloud LLM → cost tracking logged
- Multiple providers tested (OpenAI, Anthropic, Groq)

**E2E Tests:**
- Configure cloud LLM → transcribe → suggestions from cloud → cost logged

### Dependencies & Blockers

- **Dependencies:** Story 5.1 (Ollama integration)
- **Blockers:** User must provide API keys

### Acceptance Criteria

- ✅ Cloud LLM client works for multiple providers
- ✅ API keys stored securely
- ✅ Cost tracking implemented
- ✅ Automatic failover (cloud → Ollama)

---

## Story 5.3: Sliding Window Context Management

**As a** user, **I want** the AI to maintain context without hitting token limits, **so that** 2-hour meetings work.

### Tasks

- [ ] **5.3.1** Implement sliding window (last 10 min + summary)
- [ ] **5.3.2** Add token counter UI (show remaining capacity)
- [ ] **5.3.3** Automatic summarization when window slides
- [ ] **5.3.4** Test with 2-hour meeting simulation

### Subtasks (5.3.1 — Sliding Window)

```typescript
interface TranscriptWindow {
  fullTranscript: string[]; // Last 10 minutes of chunks
  summary: string; // Summarized earlier context
}

class MeetingContext {
  private chunks: string[] = [];
  private summary: string = '';
  private readonly MAX_CHUNKS = 10; // ~10 minutes (1 chunk/min)
  
  addChunk(transcript: string) {
    this.chunks.push(transcript);
    
    // If window exceeds max, summarize and slide
    if (this.chunks.length > this.MAX_CHUNKS) {
      this.slideWindow();
    }
  }
  
  private async slideWindow() {
    // Summarize oldest chunks
    const toSummarize = this.chunks.slice(0, this.MAX_CHUNKS / 2);
    const newSummary = await summarizeChunks(toSummarize);
    
    // Update summary (append new to existing)
    this.summary = this.summary 
      ? `${this.summary}\n\n**Later:** ${newSummary}`
      : newSummary;
    
    // Remove summarized chunks
    this.chunks = this.chunks.slice(this.MAX_CHUNKS / 2);
  }
  
  getContext(): string {
    const recentTranscript = this.chunks.join('\n\n');
    return this.summary 
      ? `**Earlier Summary:**\n${this.summary}\n\n**Recent Transcript:**\n${recentTranscript}`
      : recentTranscript;
  }
}
```

### Subtasks (5.3.2 — Token Counter UI)

```typescript
// Estimate tokens (rough: 1 token ≈ 4 characters)
function estimateTokens(text: string): number {
  return Math.ceil(text.length / 4);
}

function getContextStats(context: MeetingContext): {
  totalTokens: number;
  maxTokens: number;
  remainingTokens: number;
  percentUsed: number;
} {
  const contextText = context.getContext();
  const totalTokens = estimateTokens(contextText);
  const maxTokens = 8000; // Conservative estimate for most models
  
  return {
    totalTokens,
    maxTokens,
    remainingTokens: maxTokens - totalTokens,
    percentUsed: (totalTokens / maxTokens) * 100
  };
}
```

**UI Component:**
```jsx
function TokenCounter({ meetingId }) {
  const [stats, setStats] = useState(null);
  
  useEffect(() => {
    const interval = setInterval(async () => {
      const contextStats = await window.api.invoke('meeting:context-stats', meetingId);
      setStats(contextStats);
    }, 5000); // Update every 5s
    
    return () => clearInterval(interval);
  }, [meetingId]);
  
  if (!stats) return null;
  
  return (
    <div className="token-counter">
      <div className="token-bar" style={{ width: `${stats.percentUsed}%` }} />
      <span>{stats.totalTokens} / {stats.maxTokens} tokens ({stats.percentUsed.toFixed(0)}%)</span>
    </div>
  );
}
```

### Subtasks (5.3.3 — Auto-Summarization)

```typescript
async function summarizeChunks(chunks: string[]): Promise<string> {
  const combined = chunks.join('\n\n');
  
  const response = await generateSuggestionsCloud(
    combined,
    '',
    '',
    {
      endpoint: 'http://localhost:11434/v1',
      model: 'llama3.1:8b',
      apiKey: ''
    }
  );
  
  // Custom prompt for summarization
  const summaryPrompt = `Summarize the following meeting transcript in 3-5 sentences. Focus on key decisions, action items, and important context.

Transcript:
${combined}`;
  
  // ... (implementation similar to generateSuggestions)
  
  return summary;
}
```

### Subtasks (5.3.4 — 2-Hour Test)

**Test script:**
```typescript
async function simulateLongMeeting() {
  const context = new MeetingContext();
  
  // Simulate 120 chunks (2 hours, 1 chunk/min)
  for (let i = 0; i < 120; i++) {
    const fakeTranscript = `Minute ${i}: Client discusses topic ${i % 10}...`;
    context.addChunk(fakeTranscript);
    
    if (i % 10 === 0) {
      const stats = getContextStats(context);
      console.log(`Minute ${i}: ${stats.totalTokens} tokens (${stats.percentUsed.toFixed(0)}%)`);
    }
  }
  
  console.log('Final context:', context.getContext());
}
```

### Test Plans

**Unit Tests:**
- Sliding window logic works
- Token estimation accurate (±10%)
- Summarization produces concise output

**Functional Tests:**
- Context stays under token limit throughout 2-hour meeting
- Summary captures key points from earlier chunks

**Performance Tests:**
- Summarization <30s
- Token counter updates without lag

**Integration Tests:**
- Sliding window → LLM still generates relevant suggestions

**E2E Tests:**
- 2-hour simulated meeting → no crashes → context maintained

### Dependencies & Blockers

- **Dependencies:** Story 5.1 (Ollama), Story 5.2 (Cloud LLM)
- **Blockers:** None

### Acceptance Criteria

- ✅ Sliding window implemented
- ✅ Token counter UI shows capacity
- ✅ Auto-summarization works
- ✅ 2-hour meetings supported

---

# EPIC 6: Live Meeting UI

**Goal:** Split-pane UI with transcript + suggestions, triage actions

**Timeline:** Week 6-7

**Dependencies:** Epic 5 (LLM Analysis)

---

## Story 6.1: Split-Pane Meeting View

**As a** user, **I want** split-pane UI (transcript | suggestions) during meetings, **so that** I can see real-time AI guidance.

### Tasks

- [ ] **6.1.1** Design split-pane layout (responsive)
- [ ] **6.1.2** Implement live transcript view (auto-scroll)
- [ ] **6.1.3** Implement suggestions panel (with triage actions)
- [ ] **6.1.4** Add progress indicator during LLM analysis
- [ ] **6.1.5** Implement suggestion aging (dim after 5 min)

### Subtasks (6.1.1 — Layout Design)

```jsx
// MeetingView.jsx
function MeetingView({ meetingId }) {
  return (
    <div className="meeting-view">
      <div className="transcript-pane">
        <TranscriptView meetingId={meetingId} />
      </div>
      <div className="suggestions-pane">
        <SuggestionsPanel meetingId={meetingId} />
      </div>
    </div>
  );
}
```

**CSS:**
```css
.meeting-view {
  display: flex;
  height: 100vh;
  gap: 1rem;
}

.transcript-pane {
  flex: 1;
  overflow-y: auto;
  padding: 1rem;
  border-right: 1px solid #ccc;
}

.suggestions-pane {
  flex: 1;
  overflow-y: auto;
  padding: 1rem;
}
```

### Subtasks (6.1.2 — Live Transcript View)

```jsx
function TranscriptView({ meetingId }) {
  const [chunks, setChunks] = useState([]);
  const scrollRef = useRef(null);
  
  useEffect(() => {
    // Load existing chunks
    loadTranscript();
    
    // Subscribe to new chunks
    const unsubscribe = window.api.on('transcript:new-chunk', (chunk) => {
      setChunks(prev => [...prev, chunk]);
    });
    
    return () => unsubscribe();
  }, [meetingId]);
  
  useEffect(() => {
    // Auto-scroll to bottom when new chunks arrive
    if (scrollRef.current) {
      scrollRef.current.scrollTop = scrollRef.current.scrollHeight;
    }
  }, [chunks]);
  
  async function loadTranscript() {
    const result = await window.api.invoke('db:transcript-chunks:list', meetingId);
    setChunks(result);
  }
  
  return (
    <div ref={scrollRef} className="transcript-view">
      {chunks.map((chunk, i) => (
        <div key={i} className="transcript-chunk">
          <span className="timestamp">{chunk.timestamp}</span>
          <p>{chunk.text}</p>
        </div>
      ))}
    </div>
  );
}
```

### Subtasks (6.1.3 — Suggestions Panel)

```jsx
function SuggestionsPanel({ meetingId }) {
  const [suggestions, setSuggestions] = useState([]);
  
  useEffect(() => {
    loadSuggestions();
    
    const unsubscribe = window.api.on('suggestions:new', (newSuggestions) => {
      setSuggestions(prev => [...prev, ...newSuggestions]);
    });
    
    return () => unsubscribe();
  }, [meetingId]);
  
  async function loadSuggestions() {
    const result = await window.api.invoke('db:suggestions:list', meetingId);
    setSuggestions(result);
  }
  
  async function handleTriage(suggestionId, action) {
    await window.api.invoke('db:suggestions:update-status', {
      id: suggestionId,
      status: action // 'acknowledged' or 'dismissed'
    });
    
    setSuggestions(prev => 
      prev.map(s => s.id === suggestionId ? { ...s, status: action } : s)
    );
  }
  
  function copyToClipboard(text) {
    navigator.clipboard.writeText(text);
  }
  
  return (
    <div className="suggestions-panel">
      <h3>Suggestions</h3>
      {suggestions.map(s => (
        <div 
          key={s.id} 
          className={`suggestion ${s.status} ${isOld(s.timestamp) ? 'aged' : ''}`}
        >
          <span className="type">{s.type}</span>
          <p>{s.text}</p>
          <div className="actions">
            <button onClick={() => handleTriage(s.id, 'acknowledged')}>✓</button>
            <button onClick={() => handleTriage(s.id, 'dismissed')}>✗</button>
            <button onClick={() => copyToClipboard(s.text)}>📋</button>
          </div>
          <span className="timestamp">{s.timestamp}</span>
        </div>
      ))}
    </div>
  );
}
```

### Subtasks (6.1.4 — Progress Indicator)

```jsx
function SuggestionsPanel({ meetingId }) {
  const [analyzing, setAnalyzing] = useState(false);
  const [progress, setProgress] = useState(0);
  
  useEffect(() => {
    window.api.on('llm:analysis-started', () => {
      setAnalyzing(true);
      setProgress(0);
    });
    
    window.api.on('llm:analysis-progress', (percent) => {
      setProgress(percent);
    });
    
    window.api.on('llm:analysis-completed', () => {
      setAnalyzing(false);
    });
  }, []);
  
  return (
    <div className="suggestions-panel">
      {analyzing && (
        <div className="progress-indicator">
          <div className="progress-bar" style={{ width: `${progress}%` }} />
          <span>Analyzing... {progress}%</span>
        </div>
      )}
      {/* ... suggestions list */}
    </div>
  );
}
```

### Subtasks (6.1.5 — Suggestion Aging)

```typescript
function isOld(timestamp: string): boolean {
  const suggestionTime = new Date(timestamp).getTime();
  const now = Date.now();
  const ageMinutes = (now - suggestionTime) / 60000;
  return ageMinutes > 5;
}
```

**CSS:**
```css
.suggestion.aged {
  opacity: 0.5;
  background-color: #f5f5f5;
}
```

### Test Plans

**Unit Tests:**
- Transcript auto-scrolls on new chunks
- Suggestions update in real-time
- Triage actions update database

**Functional Tests:**
- Split-pane layout responsive (resizes correctly)
- Progress indicator shows during LLM analysis
- Aged suggestions dim after 5 minutes

**Integration Tests:**
- New transcript chunk → triggers LLM analysis → suggestions appear
- Copy to clipboard works

**E2E Tests:**
- Start meeting → transcript appears → suggestions appear → triage suggestion → status updates

### Dependencies & Blockers

- **Dependencies:** Story 5.3 (LLM analysis)
- **Blockers:** None

### Acceptance Criteria

- ✅ Split-pane UI works
- ✅ Live transcript auto-scrolls
- ✅ Suggestions panel shows triage actions
- ✅ Progress indicator during analysis
- ✅ Suggestion aging implemented

---

# EPIC 7: Auto-Prep Mode (Nice-to-Have)

**Goal:** Generate pre-meeting briefing (client context + suggested questions)

**Timeline:** Week 7 (if time permits, otherwise v2)

---

## Story 7.1: Pre-Meeting Briefing

### Tasks

- [ ] **7.1.1** Build briefing generation UI
- [ ] **7.1.2** Implement briefing LLM prompt
- [ ] **7.1.3** Save briefing to file + database
- [ ] **7.1.4** Display briefing before meeting starts

### Subtasks (7.1.1 — Briefing UI)

```jsx
function PreMeetingBriefing({ clientId, meetingType }) {
  const [briefing, setBriefing] = useState(null);
  const [loading, setLoading] = useState(false);
  
  async function generateBriefing() {
    setLoading(true);
    const result = await window.api.invoke('meeting:generate-briefing', {
      clientId,
      meetingType
    });
    setBriefing(result);
    setLoading(false);
  }
  
  return (
    <div className="briefing-panel">
      <h2>Pre-Meeting Briefing</h2>
      {!briefing ? (
        <button onClick={generateBriefing} disabled={loading}>
          {loading ? 'Generating...' : 'Generate Briefing'}
        </button>
      ) : (
        <div className="briefing-content">
          <h3>Client Summary</h3>
          <p>{briefing.clientSummary}</p>
          
          <h3>Recent Meetings</h3>
          <ul>
            {briefing.recentMeetings.map(m => <li key={m.id}>{m.summary}</li>)}
          </ul>
          
          <h3>Suggested Questions</h3>
          <ul>
            {briefing.suggestedQuestions.map((q, i) => <li key={i}>{q}</li>)}
          </ul>
          
          <h3>Relevant Vendors/Products</h3>
          <ul>
            {briefing.vendors.map(v => <li key={v.id}>{v.name}: {v.description}</li>)}
          </ul>
        </div>
      )}
    </div>
  );
}
```

### Subtasks (7.1.2 — Briefing Prompt)

```typescript
async function generateMeetingBriefing(clientId: number, meetingType: string): Promise<any> {
  const client = db.prepare('SELECT * FROM clients WHERE id = ?').get(clientId);
  const recentMeetings = db.prepare(`
    SELECT * FROM meetings 
    WHERE client_id = ? 
    ORDER BY start_time DESC 
    LIMIT 3
  `).all(clientId);
  
  const prompt = `Generate a pre-meeting briefing for a ${meetingType} meeting with ${client.name}.

**Client Context:**
- Industry: ${client.industry}
- Tech Stack: ${client.tech_stack_summary}
- Notes: ${client.notes}

**Recent Meetings:**
${recentMeetings.map(m => `- ${m.title} (${m.start_time}): ${m.summary}`).join('\n')}

**Generate:**
1. **Client Summary** (2-3 sentences)
2. **Suggested Questions** (5 strategic questions to ask)
3. **Relevant Vendors/Products** (from our catalog that might help this client)`;

  const response = await generateSuggestionsCloud(prompt, '', '', getCloudLLMConfig());
  
  // Parse response (assumes structured output from LLM)
  return {
    clientSummary: /* extract from response */,
    recentMeetings,
    suggestedQuestions: /* extract from response */,
    vendors: /* semantic search vendors based on client tech stack */
  };
}
```

### Test Plans

**Functional Tests:**
- Briefing generated in <30s
- Briefing includes client summary, questions, vendors

**E2E Tests:**
- Select client → generate briefing → briefing displayed → start meeting

### Dependencies & Blockers

- **Dependencies:** Story 4.1 (Client profiles), Story 5.1 (LLM)
- **Blockers:** None

### Acceptance Criteria

- ✅ Briefing generation works
- ✅ Briefing saved to file + database
- ✅ Display briefing before meeting

---

# EPIC 8: Post-Meeting Artifacts (Nice-to-Have)

**Goal:** Generate summary, action items, follow-up email draft after meeting

**Timeline:** Week 7 (if time permits, otherwise v2)

---

## Story 8.1: Artifact Generation

### Tasks

- [ ] **8.1.1** Implement post-meeting summary generation
- [ ] **8.1.2** Extract action items from transcript
- [ ] **8.1.3** Draft follow-up email (customizable)
- [ ] **8.1.4** Save artifacts to file + database

### Subtasks (8.1.1 — Summary Generation)

```typescript
async function generateMeetingSummary(meetingId: number): Promise<string> {
  const chunks = db.prepare(`
    SELECT text FROM transcript_chunks 
    WHERE meeting_id = ? 
    ORDER BY chunk_index
  `).all(meetingId);
  
  const fullTranscript = chunks.map(c => c.text).join('\n\n');
  
  const prompt = `Summarize this meeting transcript. Include:
- **Key Points:** Main topics discussed
- **Decisions:** Decisions made
- **Next Steps:** What happens next

Transcript:
${fullTranscript}`;

  const response = await generateSuggestionsCloud(prompt, '', '', getCloudLLMConfig());
  return response[0]; // Assuming single response
}
```

### Subtasks (8.1.2 — Action Items)

```typescript
async function extractActionItems(meetingId: number): Promise<Array<{ text: string; assignedTo?: string; dueDate?: string }>> {
  const chunks = db.prepare(`
    SELECT text FROM transcript_chunks 
    WHERE meeting_id = ? 
    ORDER BY chunk_index
  `).all(meetingId);
  
  const fullTranscript = chunks.map(c => c.text).join('\n\n');
  
  const prompt = `Extract action items from this transcript. For each action item, specify:
- **Action:** What needs to be done
- **Assigned To:** Who is responsible (if mentioned)
- **Due Date:** When it's due (if mentioned)

Transcript:
${fullTranscript}

Format: JSON array of objects with fields: text, assignedTo, dueDate`;

  const response = await generateSuggestionsCloud(prompt, '', '', getCloudLLMConfig());
  return JSON.parse(response[0]);
}
```

### Subtasks (8.1.3 — Follow-Up Email Draft)

```typescript
async function draftFollowUpEmail(meetingId: number): Promise<string> {
  const meeting = db.prepare('SELECT * FROM meetings WHERE id = ?').get(meetingId);
  const client = db.prepare('SELECT * FROM clients WHERE id = ?').get(meeting.client_id);
  const summary = await generateMeetingSummary(meetingId);
  const actionItems = await extractActionItems(meetingId);
  
  const prompt = `Draft a follow-up email for a meeting with ${client.name}.

**Meeting Summary:**
${summary}

**Action Items:**
${actionItems.map(a => `- ${a.text}${a.assignedTo ? ` (${a.assignedTo})` : ''}`).join('\n')}

**Email should:**
- Thank them for their time
- Summarize key points
- List action items
- Suggest next steps

Keep it professional and concise (3-4 paragraphs).`;

  const response = await generateSuggestionsCloud(prompt, '', '', getCloudLLMConfig());
  return response[0];
}
```

### Test Plans

**Functional Tests:**
- Summary generated in <60s
- Action items extracted correctly (manual review)
- Follow-up email draft is professional

**E2E Tests:**
- End meeting → generate artifacts → artifacts saved → display in UI

### Dependencies & Blockers

- **Dependencies:** Story 5.1 (LLM)
- **Blockers:** None

### Acceptance Criteria

- ✅ Summary generated after meeting
- ✅ Action items extracted
- ✅ Follow-up email draft created
- ✅ Artifacts saved to files + database

---

# EPIC 9: Settings & Configuration

**Goal:** User-configurable settings (API keys, audio, LLM, STT)

**Timeline:** Week 7

---

## Story 9.1: Settings UI

### Tasks

- [ ] **9.1.1** Design settings UI (tabs: General, Audio, STT, LLM, Advanced)
- [ ] **9.1.2** Implement General settings
- [ ] **9.1.3** Implement Audio settings (device selection, health check)
- [ ] **9.1.4** Implement STT settings (Whisper vs cloud)
- [ ] **9.1.5** Implement LLM settings (Ollama vs cloud, model selection)
- [ ] **9.1.6** Implement Advanced settings (logging, performance)

### Subtasks (9.1.1 — Settings UI)

```jsx
function SettingsView() {
  const [activeTab, setActiveTab] = useState('general');
  
  return (
    <div className="settings-view">
      <nav className="settings-tabs">
        <button onClick={() => setActiveTab('general')}>General</button>
        <button onClick={() => setActiveTab('audio')}>Audio</button>
        <button onClick={() => setActiveTab('stt')}>STT</button>
        <button onClick={() => setActiveTab('llm')}>LLM</button>
        <button onClick={() => setActiveTab('advanced')}>Advanced</button>
      </nav>
      
      <div className="settings-content">
        {activeTab === 'general' && <GeneralSettings />}
        {activeTab === 'audio' && <AudioSettings />}
        {activeTab === 'stt' && <STTSettings />}
        {activeTab === 'llm' && <LLMSettings />}
        {activeTab === 'advanced' && <AdvancedSettings />}
      </div>
    </div>
  );
}
```

### Subtasks (9.1.2 — General Settings)

```jsx
function GeneralSettings() {
  const [autoStart, setAutoStart] = useState(false);
  const [checkUpdates, setCheckUpdates] = useState(true);
  
  useEffect(() => {
    loadSettings();
  }, []);
  
  async function loadSettings() {
    const settings = await window.api.invoke('settings:get', 'general');
    setAutoStart(settings.autoStart);
    setCheckUpdates(settings.checkUpdates);
  }
  
  async function handleSave() {
    await window.api.invoke('settings:set', {
      category: 'general',
      data: { autoStart, checkUpdates }
    });
  }
  
  return (
    <div>
      <h2>General Settings</h2>
      <label>
        <input type="checkbox" checked={autoStart} onChange={(e) => setAutoStart(e.target.checked)} />
        Launch on startup
      </label>
      <label>
        <input type="checkbox" checked={checkUpdates} onChange={(e) => setCheckUpdates(e.target.checked)} />
        Check for updates automatically
      </label>
      <button onClick={handleSave}>Save</button>
    </div>
  );
}
```

### Subtasks (9.1.3 — Audio Settings)

```jsx
function AudioSettings() {
  const [devices, setDevices] = useState([]);
  const [selectedMic, setSelectedMic] = useState('');
  const [selectedOutput, setSelectedOutput] = useState('');
  const [healthStatus, setHealthStatus] = useState('');
  
  useEffect(() => {
    loadDevices();
  }, []);
  
  async function loadDevices() {
    const result = await window.api.invoke('audio:list-devices');
    setDevices(result);
  }
  
  async function testAudio() {
    setHealthStatus('Testing...');
    const result = await window.api.invoke('audio:test');
    setHealthStatus(result);
  }
  
  return (
    <div>
      <h2>Audio Settings</h2>
      
      <h3>Microphone</h3>
      <select value={selectedMic} onChange={(e) => setSelectedMic(e.target.value)}>
        {devices.filter(d => d.type === 'input').map(d => (
          <option key={d.id} value={d.id}>{d.name}</option>
        ))}
      </select>
      
      <h3>System Audio</h3>
      <select value={selectedOutput} onChange={(e) => setSelectedOutput(e.target.value)}>
        {devices.filter(d => d.type === 'output').map(d => (
          <option key={d.id} value={d.id}>{d.name}</option>
        ))}
      </select>
      
      <h3>Audio Health Check</h3>
      <button onClick={testAudio}>Test Audio</button>
      {healthStatus && <p>{healthStatus}</p>}
      
      <h3>BlackHole Setup</h3>
      <p>
        For system audio capture, install BlackHole:
        <code>brew install blackhole-2ch</code>
      </p>
      <a href="https://existential.audio/blackhole/" target="_blank">BlackHole Documentation</a>
    </div>
  );
}
```

### Subtasks (9.1.4 — STT Settings)

```jsx
function STTSettings() {
  const [provider, setProvider] = useState('whisper');
  const [apiKey, setAPIKey] = useState('');
  const [endpoint, setEndpoint] = useState('');
  
  useEffect(() => {
    loadSettings();
  }, []);
  
  async function loadSettings() {
    const settings = await window.api.invoke('settings:get', 'stt');
    setProvider(settings.provider);
    setAPIKey(settings.apiKey || '');
    setEndpoint(settings.endpoint || '');
  }
  
  async function handleSave() {
    await window.api.invoke('settings:set', {
      category: 'stt',
      data: { provider, apiKey, endpoint }
    });
  }
  
  return (
    <div>
      <h2>STT Settings</h2>
      
      <h3>Provider</h3>
      <label>
        <input type="radio" value="whisper" checked={provider === 'whisper'} onChange={(e) => setProvider(e.target.value)} />
        Whisper (local, GPU-accelerated)
      </label>
      <label>
        <input type="radio" value="cloud" checked={provider === 'cloud'} onChange={(e) => setProvider(e.target.value)} />
        Cloud (OpenAI-compatible)
      </label>
      
      {provider === 'cloud' && (
        <>
          <h3>Cloud STT Configuration</h3>
          <input type="text" placeholder="API Endpoint" value={endpoint} onChange={(e) => setEndpoint(e.target.value)} />
          <input type="password" placeholder="API Key" value={apiKey} onChange={(e) => setAPIKey(e.target.value)} />
        </>
      )}
      
      <button onClick={handleSave}>Save</button>
    </div>
  );
}
```

### Subtasks (9.1.5 — LLM Settings)

```jsx
function LLMSettings() {
  const [provider, setProvider] = useState('ollama');
  const [model, setModel] = useState('llama3.1:8b');
  const [apiKey, setAPIKey] = useState('');
  const [endpoint, setEndpoint] = useState('');
  
  return (
    <div>
      <h2>LLM Settings</h2>
      
      <h3>Provider</h3>
      <label>
        <input type="radio" value="ollama" checked={provider === 'ollama'} onChange={(e) => setProvider(e.target.value)} />
        Ollama (local)
      </label>
      <label>
        <input type="radio" value="cloud" checked={provider === 'cloud'} onChange={(e) => setProvider(e.target.value)} />
        Cloud (OpenAI, Anthropic, Groq, etc.)
      </label>
      
      {provider === 'ollama' && (
        <>
          <h3>Ollama Configuration</h3>
          <select value={model} onChange={(e) => setModel(e.target.value)}>
            <option value="llama3.1:8b">Llama 3.1 8B (fast)</option>
            <option value="mixtral:8x7b">Mixtral 8x7B (slower, better)</option>
          </select>
          <p>Ollama must be running: <code>ollama serve</code></p>
        </>
      )}
      
      {provider === 'cloud' && (
        <>
          <h3>Cloud LLM Configuration</h3>
          <select value={endpoint} onChange={(e) => setEndpoint(e.target.value)}>
            <option value="https://api.openai.com/v1">OpenAI</option>
            <option value="https://api.anthropic.com/v1">Anthropic</option>
            <option value="https://api.groq.com/openai/v1">Groq</option>
            <option value="https://openrouter.ai/api/v1">OpenRouter</option>
          </select>
          <input type="password" placeholder="API Key" value={apiKey} onChange={(e) => setAPIKey(e.target.value)} />
        </>
      )}
      
      <button onClick={handleSave}>Save</button>
    </div>
  );
}
```

### Subtasks (9.1.6 — Advanced Settings)

```jsx
function AdvancedSettings() {
  const [suggestionFrequency, setSuggestionFrequency] = useState(60);
  const [contextWindowSize, setContextWindowSize] = useState(10);
  const [verboseLogging, setVerboseLogging] = useState(false);
  
  return (
    <div>
      <h2>Advanced Settings</h2>
      
      <h3>Suggestion Frequency</h3>
      <input 
        type="number" 
        min="30" 
        max="120" 
        value={suggestionFrequency} 
        onChange={(e) => setSuggestionFrequency(parseInt(e.target.value))} 
      />
      <span>seconds between LLM analysis cycles</span>
      
      <h3>Context Window Size</h3>
      <input 
        type="number" 
        min="5" 
        max="20" 
        value={contextWindowSize} 
        onChange={(e) => setContextWindowSize(parseInt(e.target.value))} 
      />
      <span>minutes of recent transcript to keep in context</span>
      
      <h3>Logging</h3>
      <label>
        <input type="checkbox" checked={verboseLogging} onChange={(e) => setVerboseLogging(e.target.checked)} />
        Enable verbose logging (for debugging)
      </label>
      
      <button onClick={handleSave}>Save</button>
    </div>
  );
}
```

### Test Plans

**Functional Tests:**
- Settings saved → persist across restarts
- Device selection works
- API keys stored securely

**E2E Tests:**
- Change LLM provider → restart app → new provider used

### Dependencies & Blockers

- **Dependencies:** Story 1.5 (IPC), all previous epics (integrations)
- **Blockers:** None

### Acceptance Criteria

- ✅ Settings UI works
- ✅ All settings categories implemented
- ✅ Settings persist across restarts

---

# EPIC 10: Health & Diagnostics

**Goal:** System health dashboard, safe mode, graceful degradation

**Timeline:** Week 7

---

## Story 10.1: System Health Dashboard

### Tasks

- [ ] **10.1.1** Build health check panel in settings
- [ ] **10.1.2** Implement subsystem health checks (audio, STT, LLM, DB)
- [ ] **10.1.3** Add latency metrics dashboard
- [ ] **10.1.4** Implement "Test Audio" and "Test LLM" buttons

### Subtasks (10.1.1-10.1.4 — Implementation detailed earlier in Epic 10)

[Already detailed in earlier sections — refer to Story 10.1 from the main epics document]

---

## Story 10.2: Graceful Degradation & Safe Mode

### Tasks

- [ ] **10.2.1** Implement graceful degradation for each subsystem
- [ ] **10.2.2** Add safe mode launch (--safe-mode CLI flag)
- [ ] **10.2.3** Implement fallback workflows (e.g., import audio if capture fails)
- [ ] **10.2.4** Add comprehensive error logging

[Details provided earlier in Epic 10]

---

# EPIC 11: Packaging & Distribution

**Goal:** Build macOS app bundle, code signing, auto-updater

**Timeline:** Week 8

---

## Story 11.1: macOS App Bundle

### Tasks

- [ ] **11.1.1** Configure electron-builder for macOS
- [ ] **11.1.2** Bundle Rust addon (universal binary: ARM64 + x86_64)
- [ ] **11.1.3** Bundle Whisper.cpp binary
- [ ] **11.1.4** Code signing + notarization (for macOS Gatekeeper)
- [ ] **11.1.5** Configure auto-updater (GitHub releases)
- [ ] **11.1.6** Test installation on clean Mac

### Subtasks (11.1.1 — electron-builder Config)

**electron-builder.json:**
```json
{
  "appId": "com.pocketconsigliere.app",
  "productName": "Pocket Consigliere",
  "directories": {
    "output": "dist"
  },
  "files": [
    "dist/**/*",
    "resources/**/*",
    "!**/*.ts",
    "!**/*.map"
  ],
  "mac": {
    "target": ["dmg", "zip"],
    "category": "public.app-category.productivity",
    "icon": "resources/icon.icns",
    "hardenedRuntime": true,
    "gatekeeperAssess": false,
    "entitlements": "entitlements.mac.plist",
    "entitlementsInherit": "entitlements.mac.plist"
  },
  "dmg": {
    "contents": [
      {
        "x": 130,
        "y": 220
      },
      {
        "x": 410,
        "y": 220,
        "type": "link",
        "path": "/Applications"
      }
    ]
  }
}
```

### Subtasks (11.1.2 — Bundle Rust Addon)

**Build universal binary:**
```bash
# Build for ARM64
cargo build --release --target aarch64-apple-darwin

# Build for x86_64
cargo build --release --target x86_64-apple-darwin

# Create universal binary
lipo -create \
  target/aarch64-apple-darwin/release/libaudio.dylib \
  target/x86_64-apple-darwin/release/libaudio.dylib \
  -output resources/audio.darwin-universal.node
```

**electron-builder config:**
```json
{
  "extraResources": [
    { "from": "resources/audio.darwin-universal.node", "to": "audio.node" }
  ]
}
```

### Subtasks (11.1.3 — Bundle Whisper)

```json
{
  "extraResources": [
    { "from": "whisper.cpp/main", "to": "whisper" },
    { "from": "whisper.cpp/models/ggml-base.en.bin", "to": "models/ggml-base.en.bin" }
  ]
}
```

### Subtasks (11.1.4 — Code Signing + Notarization)

**Entitlements (entitlements.mac.plist):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>com.apple.security.cs.allow-jit</key>
  <true/>
  <key>com.apple.security.cs.allow-unsigned-executable-memory</key>
  <true/>
  <key>com.apple.security.device.audio-input</key>
  <true/>
  <key>com.apple.security.files.user-selected.read-write</key>
  <true/>
</dict>
</plist>
```

**Sign + notarize:**
```bash
# Sign
codesign --force --deep --sign "Developer ID Application: Your Name" dist/mac/Pocket\ Consigliere.app

# Notarize
xcrun notarytool submit dist/mac/Pocket\ Consigliere.dmg --keychain-profile "AC_PASSWORD" --wait

# Staple
xcrun stapler staple dist/mac/Pocket\ Consigliere.dmg
```

### Subtasks (11.1.5 — Auto-Updater)

**electron-builder config:**
```json
{
  "publish": {
    "provider": "github",
    "owner": "your-org",
    "repo": "pocket-consigliere"
  }
}
```

**Main process:**
```typescript
import { autoUpdater } from 'electron-updater';

autoUpdater.checkForUpdatesAndNotify();

autoUpdater.on('update-available', () => {
  dialog.showMessageBox({
    type: 'info',
    title: 'Update Available',
    message: 'A new version is available. It will be downloaded in the background.'
  });
});

autoUpdater.on('update-downloaded', () => {
  dialog.showMessageBox({
    type: 'info',
    title: 'Update Ready',
    message: 'Update downloaded. Restart to install.',
    buttons: ['Restart', 'Later']
  }).then((result) => {
    if (result.response === 0) {
      autoUpdater.quitAndInstall();
    }
  });
});
```

### Subtasks (11.1.6 — Test Installation)

**Test checklist:**
- [ ] Download DMG on clean Mac (no dev tools)
- [ ] Install app (drag to Applications)
- [ ] Launch app (verify no Gatekeeper warning)
- [ ] Test audio capture (BlackHole must be installed separately)
- [ ] Test STT (Whisper bundled)
- [ ] Test LLM (Ollama must be installed separately)
- [ ] Test database (creates files in ~/Library/Application Support/)
- [ ] Test auto-updater (mock release on GitHub)

### Test Plans

**Functional Tests:**
- App builds without errors
- Universal binary works on ARM64 + x86_64 Macs
- Code signing valid
- Notarization succeeds

**E2E Tests:**
- Install on clean Mac → launch → all features work
- Auto-updater downloads and installs update

### Dependencies & Blockers

- **Dependencies:** All previous epics (everything must be working)
- **Blockers:** Apple Developer account (for code signing + notarization)

### Acceptance Criteria

- ✅ macOS app bundle created
- ✅ Universal binary (ARM64 + x86_64)
- ✅ Code signed + notarized
- ✅ Auto-updater configured
- ✅ Installation tested on clean Mac

---

# Summary: Timeline & Milestones

| Week | Epic | Deliverable |
|------|------|-------------|
| **1** | **Epic 0** | **Week 1 POC: Audio addon + Whisper + Ollama + minimal UI (GO/NO-GO DECISION)** |
| 2 | Epic 1 | Foundation (Electron, SQLite, Vector DB, IPC) |
| 3 | Epic 2 | Audio Capture (Rust addon, device selection, health check) |
| 4 | Epic 3 | Speech-to-Text (Whisper.cpp + cloud fallback) |
| 4-5 | Epic 4 | Knowledge Base (Clients, Vendors) |
| 5-6 | Epic 5 | LLM Analysis Engine (Ollama + cloud + sliding window) |
| 6-7 | Epic 6 | Live Meeting UI (split-pane, suggestions, triage) |
| 7 | Epic 7 | Auto-Prep Mode (nice-to-have) |
| 7 | Epic 8 | Post-Meeting Artifacts (nice-to-have) |
| 7 | Epic 9 | Settings & Config |
| 7 | Epic 10 | Health & Diagnostics |
| 8 | Epic 11 | Packaging & Distribution |

**Total: 7-8 weeks from Week 1 POC to shippable V1**

---

# AI Assistance Summary

**Where AI helps throughout:**
- Week 1: "Convert Rust audio code to napi-rs bindings"
- Week 2-3: "Generate Electron boilerplate + React CRUD components"
- Week 4-5: "Write semantic search backend for vector DB"
- Week 5-6: "Optimize LLM prompt for consultant copilot"
- Week 6-7: "Generate React split-pane UI with real-time updates"
- Week 7-8: "Debug Rust compiler errors" + "Configure electron-builder"

**AI debugging pattern:**
- Paste error → AI explains → suggests fix → apply → test → repeat

---

**End of Implementation Plan v2**
