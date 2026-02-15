# Pocket Consigliere - AI Assistant Instructions

**Generated:** 2026-02-15  
**Planning Session:** [Hybrid Architecture Discussion](transcripts/2026-02-15-hybrid-architecture-discussion.md)  
**For:** Claude Code, Cursor, GitHub Copilot  

---

## Project Overview

Pocket Consigliere is a **real-time AI meeting copilot** for high-end consultants. It captures system audio + microphone during client meetings, transcribes in real-time using Whisper, and sends rolling transcript context to a reasoning LLM every ~60 seconds. The LLM returns strategic suggestions (questions, notes, talking points, research topics) in a sidebar/overlay UI.

**Key goals:**
- Provide strategic guidance during client meetings without visible prompts or delays
- Maintain client profiles, vendor catalog, and historical meeting context
- Generate post-meeting artifacts (transcript, summary, action items, briefing)
- Fully local-first (privacy, no cloud dependency except optional LLM API)

**Target users:** Jonathan & Roy (v1), later high-end consultants talking to senior client stakeholders

**Success criteria:** AI that feels like a brilliant, silent partner who's done the research and surfaces relevant insights in real-time

---

## Architecture Summary

**Approach:** Hybrid architecture (Electron + Rust audio addon)

**Key decisions:**
- **Electron + Rust audio addon (napi-rs)** — NOT pure Node.js (audio unproven) or full Tauri (Rust learning curve for entire backend). Use Meetily's proven Rust audio code compiled as Node addon, keep business logic in Node.js.
- **Ollama (local) + Cloud APIs (configurable)** — Support both from day 1. Ollama for privacy/cost (free, local), cloud for speed/quality (paid, faster).
- **Realistic performance targets** — STT <10s (GPU-accelerated Whisper), LLM <60s (Ollama with context), total cycle <90s. Not aggressive, achievable on M1/M2 Mac.

**Critical constraints:**
- macOS-only for v1 (Windows/Linux in v2 after cross-platform validation)
- 2-user tool (Jonathan + Roy), no multi-user features in v1
- Fully local (except cloud API calls if configured)
- Self-contained .app bundle (no server dependencies)

**Non-negotiables:**
- Week 1 POC must validate audio capture + Whisper + Ollama chain (go/no-go decision)
- Audio addon must be from Meetily (proven, MIT licensed) — don't reinvent
- Commit and push after every Epic (checkpoints)

---

## Tech Stack

**Frontend:**
- **Electron** - App shell (NOT Tauri — avoid Rust learning curve for UI/business logic)
- **React + TypeScript** - UI framework
- **Vite** - Dev server + hot reload

**Backend:**
- **Node.js + TypeScript** - Business logic (LLM client, database, IPC)
- **Rust addon (napi-rs)** - Audio capture only (fork Meetily's audio module)
- **SQLite (better-sqlite3)** - Structured data (clients, vendors, meetings, transcripts)
- **LanceDB** - Vector embeddings for semantic search

**Audio:**
- **Rust addon (`cpal` crate)** - Cross-platform audio I/O (forked from Meetily)
- **BlackHole** - macOS virtual audio driver (user-installed, required for system audio capture)

**STT:**
- **Whisper.cpp (Metal-accelerated)** - Local transcription (<10s for 60s audio on GPU)
- **OpenAI-compatible cloud STT** - Fallback if Whisper too slow

**LLM:**
- **Ollama** - Local LLM inference (Llama 3.1 8B or Mixtral 8x7B)
- **OpenAI-compatible cloud APIs** - Claude, GPT-4, Groq, OpenRouter (configurable)

**Embeddings:**
- **all-MiniLM-L6-v2** - Local embedding model (fast, small, no API)

**Special components:**
- **Rust audio addon** - Fork Meetily's audio module, compile via napi-rs to `.node` binary
- **Sliding window context** - Last 10 min full transcript + summarized earlier context (prevents token overflow)

**Dependencies:**
- **Meetily audio module** (GitHub: Zackriya-Solutions/meeting-minutes, MIT license)
- **Whisper.cpp** (ggerganov/whisper.cpp, Metal support for macOS GPU)
- **Ollama** (local LLM server, user must install separately)

---

## Epic Sequence (Build Order)

**IMPORTANT:** Build in this order. Dependencies are critical.

| Epic | Priority | Why First | Blockers |
|------|----------|-----------|----------|
| **Epic 0: Week 1 POC** | **CRITICAL** | **Go/no-go validation:** Audio addon + Whisper + Ollama chain must work | None |
| Epic 1: Foundation | Must-have | Electron scaffold, SQLite, Vector DB, IPC — everything else depends on this | Epic 0 |
| Epic 2: Audio Capture (Rust Addon) | Must-have | Audio is highest risk, must work before STT | Epic 0, Epic 1 |
| Epic 3: Speech-to-Text | Must-have | Needs audio capture functional | Epic 2 |
| Epic 4: Knowledge Base | Must-have | Client/vendor data needed for LLM context | Epic 1 |
| Epic 5: LLM Analysis Engine | Must-have | Needs STT + Knowledge Base | Epic 3, Epic 4 |
| Epic 6: Live Meeting UI | Must-have | Displays transcript + suggestions from LLM | Epic 5 |
| Epic 7: Auto-Prep Mode | Nice-to-have | Pre-meeting briefing (deferred if time-constrained) | Epic 4, Epic 5 |
| Epic 8: Post-Meeting Artifacts | Nice-to-have | Summary, action items, follow-up email (deferred if time-constrained) | Epic 5 |
| Epic 9: Settings & Config | Must-have | User needs to configure API keys, audio devices | Epic 1 |
| Epic 10: Health & Diagnostics | Must-have | Operational safeguards (safe mode, health checks) | All previous epics |
| Epic 11: Packaging & Distribution | Must-have | macOS app bundle, code signing, auto-updater | All previous epics |

**After each Epic:**
- ✅ Run tests for that Epic (unit, functional, integration)
- ✅ Commit and push: `git commit -m "Epic [N]: [Name] - [Summary]"`
- ✅ Ask: "Epic [N] complete. Ready for Epic [N+1]?"

**If Week 1 POC (Epic 0) fails:**
- Pivot to "import audio file" workflow (manual recording, no live capture)
- Reassess architecture (consider cloud STT only, skip Whisper)

---

## Critical Gotchas (Things That Will Break)

### 1. Audio Capture is a Minefield
**Problem:** System audio + mic capture is OS-dependent hell. macOS requires screen recording permissions (scary dialog), audio loopback (BlackHole), breaks across OS updates.

**Mitigation:**
- ✅ **Use Meetily's Rust audio code** (fork from GitHub: Zackriya-Solutions/meeting-minutes, MIT license)
- ✅ Week 1 POC: Build audio addon FIRST, validate before building anything else
- ✅ If Week 1 POC fails, pivot to "import audio file" workflow

**Where it matters:** Epic 0 (Week 1 POC), Epic 2 (Audio Capture)

### 2. LLM Context Window Management
**Problem:** 60s transcript chunks → cumulative context → token explosion within 30 minutes. 30-min meeting = ~6,000 tokens (transcript alone), climbing to 20,000+ for 90-min calls.

**Mitigation:**
- ✅ Sliding window: last 10 min full transcript + summarized earlier context
- ✅ Token counter UI (show remaining capacity in real-time)
- ✅ Test with 2-hour meeting simulation before shipping
- ✅ Max meeting length: 2 hours (graceful degradation beyond)

**Where it matters:** Epic 5 (LLM Analysis Engine), Story 5.3 (Sliding Window)

### 3. Latency Perception = UX Killer
**Problem:** 60s LLM analysis cycle feels slow. Users will think it's broken. Timing: STT 5-15s + LLM 10-30s = 15-45s lag between speech and suggestion.

**Mitigation:**
- ✅ Progress indicator during LLM call ("Analyzing... 15s remaining")
- ✅ Show "last analyzed" timestamp on each suggestion batch
- ✅ Performance target: LLM analysis <60s (timeout at 90s)
- ✅ Suggestion aging: dim suggestions after 5 min (visual feedback)

**Where it matters:** Epic 6 (Live Meeting UI), Epic 5 (LLM Engine)

### 4. Cross-Platform Audio is a One-Way Door
**Problem:** Committing to Electron + Rust addon. Hard to reverse later.

**Mitigation:**
- ✅ Week 1 POC validates feasibility BEFORE full commitment
- ✅ Rust addon is isolated (can replace with Node-native in v2)
- ✅ If Windows/Linux support fails in v2, we've only committed to macOS

**Where it matters:** Epic 0 (Week 1 POC — this is the decision point)

### 5. Whisper.cpp Compilation (Metal Support)
**Problem:** Whisper.cpp must be compiled with Metal support for GPU acceleration. Default build is CPU-only (too slow).

**Mitigation:**
- ✅ Build command: `WHISPER_METAL=1 make` (in whisper.cpp directory)
- ✅ Test: Transcribe 60s audio, measure latency (target: <10s)
- ✅ Fallback: CPU-only if GPU unavailable (warn user about latency)

**Where it matters:** Epic 3 (Speech-to-Text), Story 3.1 (Whisper Integration)

### 6. Ollama Must Be Running
**Problem:** Ollama is a separate server process. If not running, LLM calls fail.

**Mitigation:**
- ✅ Health check on app launch: ping Ollama endpoint, show status
- ✅ Instructions in settings: "Start Ollama with `ollama serve`"
- ✅ Automatic fallback to cloud LLM if Ollama unreachable

**Where it matters:** Epic 5 (LLM Analysis Engine), Epic 10 (Health Dashboard)

---

## Test Strategy

**Test after each Epic:**
- **Unit tests:** Module-level (functions, classes)
- **Functional tests:** Feature-level (does audio capture work? does STT work?)
- **Integration tests:** Cross-module (audio → STT pipeline, STT → LLM pipeline)

**Test at end (after all Epics):**
- **Full test suite:** All unit + functional + integration tests
- **First run validation:** `npm run dev` (catches config issues tests miss)
- **Manual QA:** Smoke test critical paths (record meeting → transcribe → suggestions appear)

**Performance targets:**
- STT latency: <10s for 60s audio (GPU-accelerated Whisper)
- LLM latency: <60s for analysis (Ollama with context)
- Total cycle time: <90s (audio → STT → LLM → UI)
- Max meeting length: 2 hours (graceful degradation beyond)

**Test coverage goals:**
- Business logic: >80%
- Audio addon (Rust): Unit tests only (integration tested via functional tests)

---

## File Structure

**Key directories:**
```
pocket-consigliere/
├── src/
│   ├── main/          # Electron main process (Node.js)
│   ├── renderer/      # React UI
│   ├── shared/        # Shared types (IPC contracts)
│   └── rust-addon/    # Audio capture (Rust, compiled to .node)
├── tests/             # All tests (unit, functional, integration, E2E)
├── resources/         # Bundled binaries (Whisper, audio addon)
├── docs/              # Planning documents (see below)
└── package.json
```

**Where to find things:**
- Requirements: [pocket-consigliere-requirements.md](pocket-consigliere-requirements.md)
- Architecture: [ARCHITECTURE.md](ARCHITECTURE.md) (to be created)
- Implementation plan: [pocket-consigliere-implementation-plan-v2.md](pocket-consigliere-implementation-plan-v2.md)
- Epics & stories: [epics-stories-v2.md](epics-stories-v2.md)
- Risk analysis: [risk-considerations.md](risk-considerations.md)
- Market comparison: [market-compare-and-contrast.md](market-compare-and-contrast.md)

---

## Development Workflow

### 1. Start Building
```bash
# Clone repo
git clone https://github.com/aquaregiaswarm-blip/pocket-consigliere.git
cd pocket-consigliere

# Install dependencies
npm install

# Set up environment (if needed)
# (No .env file required for v1 — local-first)
```

### 2. Week 1 POC (Epic 0) — CRITICAL PATH
**Do this FIRST. If it fails, reassess architecture.**

```bash
# Day 1-2: Fork Meetily audio module
git clone https://github.com/Zackriya-Solutions/meeting-minutes meetily
cd meetily/src-tauri/src/audio/
# Copy relevant files to pocket-consigliere/src/rust-addon/

# Day 3-4: Build audio addon
cd pocket-consigliere/src/rust-addon/
npm install -g @napi-rs/cli
napi new pocket-consigliere-audio
# Adapt Meetily code to napi-rs bindings
cargo build --release

# Day 5: Test audio capture
node test-audio-capture.js  # 60s recording, verify mic + system audio

# Day 6: Integrate Whisper.cpp
cd ../..
git clone https://github.com/ggerganov/whisper.cpp
cd whisper.cpp
WHISPER_METAL=1 make
bash ./models/download-ggml-model.sh base.en
./main -m models/ggml-base.en.bin -f ../test-audio.wav  # Test transcription

# Day 7: Integrate Ollama
ollama serve  # (in separate terminal)
ollama pull llama3.1:8b
node test-ollama-client.js  # Test LLM analysis

# End of Week 1: E2E test
node test-poc-e2e.js  # 30-min simulated meeting (audio → STT → LLM → UI)
```

**Go/No-Go Decision:** If POC succeeds, proceed to Epic 1. If fails, pivot to "import audio file" workflow.

### 3. Implement Epic
- Read Epic goals from [implementation plan](pocket-consigliere-implementation-plan-v2.md)
- Implement tasks → subtasks
- Write tests alongside code (in `tests/` directory)
- Run tests for completed Epic: `npm test -- --Epic[N]`

### 4. Checkpoint
```bash
git add .
git commit -m "Epic [N]: [Epic Name] - [Brief summary]"
git push
```

### 5. Validation (After All Epics)
```bash
npm test           # Run full test suite
npm run dev        # First run validation (catches config issues)
```

---

## Clarifying Questions (Common Ambiguities)

**If you're unsure about:**
- **"Should I start with Epic 0 or Epic 1?"** → Start with Epic 0 (Week 1 POC). It's the go/no-go validation.
- **"Should I use TypeScript strict mode?"** → Yes. Enable strict mode in tsconfig.json.
- **"Should I bundle the Rust addon now or defer?"** → Bundle during Epic 2 (Audio Capture). Compile universal binary (ARM64 + x86_64) for macOS.
- **"Should I write tests during Epic implementation or after?"** → During. Write tests alongside code, run after Epic completes.
- **"What if Whisper is too slow on my machine?"** → Test with 60s audio. If >10s, fall back to cloud STT. Document this in Epic 3.
- **"What if Ollama doesn't fit in RAM?"** → Use Llama 3.1 8B (not Mixtral 8x7B). 8B model fits in 8 GB RAM.

**When in doubt:**
- Check [implementation plan](pocket-consigliere-implementation-plan-v2.md) for detailed subtasks
- Refer to [risk analysis](risk-considerations.md) for known gotchas
- Ask developer before making assumptions

---

## Success Checklist

Before marking project complete:
- [ ] All Epics implemented (check [epics-stories-v2.md](epics-stories-v2.md))
- [ ] Full test suite passes (`npm test`)
- [ ] App runs successfully (`npm run dev`)
- [ ] Performance targets met:
  - [ ] STT <10s for 60s audio
  - [ ] LLM <60s for analysis
  - [ ] Total cycle <90s
- [ ] Manual QA completed:
  - [ ] Record 30-min test meeting
  - [ ] Transcription accurate (>95%)
  - [ ] Suggestions relevant and non-redundant
  - [ ] UI responsive (<100ms)
- [ ] Documentation updated:
  - [ ] README with setup instructions
  - [ ] User guide (BlackHole setup, Ollama installation)
- [ ] No critical TODOs left in codebase

---

## Related Documents

- [Requirements](pocket-consigliere-requirements.md)
- [Epics & Stories v2](epics-stories-v2.md)
- [Implementation Plan v2](pocket-consigliere-implementation-plan-v2.md)
- [Risk Analysis](risk-considerations.md)
- [Market Comparison](market-compare-and-contrast.md)
- [Transcripts](transcripts/) — Planning session discussions

---

## Notes for AI Assistants

**Plan mode recommended:** This project has clear Epic sequencing with dependencies. Use plan mode to respect build order.

**Commit frequency:** Commit after every Epic (not every subtask). This creates clear checkpoints and makes code review easier.

**Test execution:** Write tests during Epic implementation. Run tests for that Epic after completion. Run full test suite at end.

**First run validation:** After all tests pass, suggest running the app (`npm run dev`). This catches config issues tests miss.

**Error handling:** If errors occur, diagnose and fix autonomously when possible. For Week 1 POC, escalate if audio capture fundamentally doesn't work (pivot decision).

**Week 1 POC is critical:** This is the highest-risk Epic. If audio addon doesn't work, the entire project approach must change. Don't proceed past Week 1 without successful validation.

---

**Generated from planning session:** 2026-02-15  
**Last updated:** 2026-02-15  
**Status:** Ready for implementation
