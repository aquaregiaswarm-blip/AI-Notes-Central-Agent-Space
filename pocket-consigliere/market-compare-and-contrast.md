# Pocket Consigliere — Market Comparison & Competitive Analysis

**Version:** 0.1  
**Date:** 2026-02-15  
**Author:** Clarice (Market Research)  
**Status:** Planning / Pre-Implementation  

---

## Purpose

This document analyzes competing products and open source alternatives to understand:
1. What has been built by others
2. What we can learn while keeping our unique requirements
3. How Pocket Consigliere differentiates
4. Technical patterns we can steal/adapt

---

## Market Landscape

The meeting assistant market splits into three categories:
1. **Hardware Products** (wearable AI note takers)
2. **Cloud SaaS** (Otter, Fireflies, Fathom, etc.)
3. **Open Source / Local-First** (Meetily, Buzz, etc.)

---

## Hardware Products (Wearable AI Note Takers)

### Fieldy (fieldy.ai)

**What it is:**
- Wearable device (clips to clothing) with high-quality mic
- Desktop app for online meetings
- Records conversations → auto-transcribes → summaries + reminders
- 3-day battery life, USB-C charging
- Privacy-focused ("Privacy is a fundamental human right")

**Key features:**
- Hands-free capture (no phone/laptop needed)
- AI-powered smart reminders from your own words
- 50+ languages supported
- Conversations organized by AI
- Desktop app syncs with wearable

**Pricing:**
- ~$200 hardware + subscription model

**What we can learn:**
- ✅ **Auto-reminder extraction** from conversations (action items → reminders)
- ✅ **Hands-free UX** (they've solved "how do I record without pulling out my phone")
- ✅ **Privacy messaging** (how they position privacy as a feature, not a limitation)

**How it's different from Pocket Consigliere:**
- ❌ Hardware dependency (you need the wearable)
- ❌ Cloud-based (sync across devices)
- ❌ Not real-time strategic suggestions during meeting
- ❌ No client context system

---

### Plaud (plaud.ai)

**What it is:**
- Wearable AI note taker (Plaud Note, Plaud NotePin)
- 2.9mm thin, magnetic/clip attachment
- 4 MEMS mics for studio-grade audio
- Transcribes 112 languages
- Speaker labels, multidimensional summaries

**Key features:**
- "Ask Plaud" (chat with your notes)
- Multimodal input (audio, highlights, text, images)
- ISO 27001, GDPR, HIPAA, SOC2 compliant
- Auto-switches between call and in-person recording
- Noise isolation AI

**Pricing:**
- ~$200 hardware + subscription model

**What we can learn:**
- ✅ **Multimodal input** (audio + images labeled as evidence)
- ✅ **Compliance positioning** (they lead with certifications for enterprise trust)
- ✅ **Auto-detection** (automatically detects call vs in-person meeting)
- ✅ **"Ask your notes" feature** (chat interface for retrieval)

**How it's different from Pocket Consigliere:**
- ❌ Hardware dependency
- ❌ Cloud-based (with compliance certs)
- ❌ Post-meeting analysis only (not real-time suggestions)
- ❌ No vendor catalog / client context system

---

## Cloud SaaS Products

### Otter.ai (Most Similar to Our Use Case)

**What it is:**
- Real-time transcription during meetings
- AI-generated summaries, action items, key points
- Integrates with Zoom, Teams, Google Meet (joins as bot)
- Speaker identification
- Shared team workspace

**Key features:**
- Cloud-based, joins meeting as participant (not system audio capture)
- Post-meeting summaries
- Team collaboration features
- CRM integrations

**Pricing:**
- Free tier available
- Pro: ~$10/user/month
- Business: ~$20/user/month

**What we can learn:**
- ✅ **UX pattern:** Split view (transcript left, insights right) works well
- ✅ **Action items extraction:** Post-meeting artifact generation is table stakes
- ✅ **Speaker ID:** Users expect this (we have it in V2, but test if V1 feels broken without it)

**How it's different from Pocket Consigliere:**
- ❌ Cloud-based SaaS (vendor lock-in, recurring subscription)
- ❌ Joins as bot participant (visible to clients)
- ❌ No real-time strategic suggestions during meeting
- ❌ No consultant-specific features (vendor catalog, client context)

---

### Fireflies.ai

**What it is:**
- Similar to Otter (transcription, summaries, action items)
- CRM integration (Salesforce, HubSpot)
- Conversation intelligence (tracks topics, sentiment)

**How it's different:**
- ❌ Cloud-based, bot joins meetings
- ❌ No real-time suggestions
- ❌ No client context system

---

### Fathom, Sembly.ai, Avoma

**Summary:** All follow the same pattern:
- Cloud-based SaaS
- Bot joins meeting as participant
- Post-meeting summaries only
- Team collaboration features
- CRM integrations

**None offer:**
- Real-time strategic suggestions
- Client intelligence system
- Vendor catalog
- Consultant-specific features

---

## Open Source / Local-First Projects

### 🏆 Meetily (GitHub: Zackriya-Solutions/meeting-minutes) — **Most Similar to Our Requirements**

**Stars:** 9,823 ⭐  
**License:** MIT  

**What it is:**
- Privacy-first AI meeting assistant
- **100% local processing** (Rust + Tauri + Next.js)
- Real-time transcription (Whisper or Parakeet, 4x faster)
- Speaker diarization
- AI summaries (Ollama local, or Claude/Groq/OpenRouter API)
- macOS, Windows, Linux

**Key features:**
- Local-first: all data stays on your machine
- GPU acceleration (Metal, CUDA, Vulkan)
- Flexible AI provider (Ollama, Claude, Groq, custom OpenAI endpoint)
- Professional audio mixing (mic + system audio, ducking, clipping prevention)
- Custom summary templates (PRO version)
- Auto-detect meetings (PRO version)

**Architecture:**
- **Backend:** Rust (fast, safe, cross-platform)
- **Frontend:** Next.js (React)
- **Framework:** Tauri (alternative to Electron, smaller binaries)
- **STT:** Whisper.cpp or Parakeet (ONNX) — both local, GPU-accelerated
- **LLM:** Ollama (local) or API providers
- **Audio:** Native audio APIs (borrowed from screenpipe)

**What we can learn from Meetily:**
- ✅ **Audio capture solved** (they've battle-tested mic + system audio on macOS/Windows/Linux)
- ✅ **GPU acceleration patterns** (Metal for macOS, CUDA for Windows)
- ✅ **Tauri vs Electron tradeoff** (Rust backend, lighter than Electron)
- ✅ **Local LLM integration** (Ollama support for fully offline operation)
- ✅ **Audio mixing architecture** (intelligent ducking, clipping prevention)
- ✅ **Custom OpenAI endpoint support** (exactly what we want for STT/LLM flexibility)

**What we should steal:**
1. **Audio capture code** (they've solved the cross-platform nightmare)
2. **GPU acceleration approach** (Metal/CUDA/Vulkan auto-detection)
3. **Ollama integration pattern** (fully local LLM inference)
4. **Tauri consideration** (lighter than Electron, but Rust learning curve)

**How it's different from Pocket Consigliere:**
- ❌ No real-time strategic suggestions (summaries are post-meeting only)
- ❌ No client context system (no profiles, vendor catalog, engagement history)
- ❌ No meeting prep mode
- ❌ No suggestion triaging (✓/✗ during meeting)

**Recommendation:** Fork Meetily's audio capture module for Pocket Consigliere's audio layer.

---

### Buzz (GitHub: chidiwilliams/buzz)

**Stars:** ~11,000 ⭐  
**License:** MIT  

**What it is:**
- Personal desktop transcription tool
- Uses OpenAI Whisper (local or API)
- Supports real-time + file transcription
- Electron-based

**What we can learn:**
- ✅ **Whisper integration patterns** (local vs API toggle)
- ✅ **Electron + native audio modules** (they've battle-tested this)
- ✅ **Permission handling** (macOS screen recording permission flow)

**How it's different:**
- ❌ No LLM analysis
- ❌ No meeting context
- ❌ Just transcription (no AI suggestions)

---

### Other Open Source Alternatives (Smaller Projects)

| Project | Stars | Key Feature | Learnings |
|---------|-------|-------------|-----------|
| **char** (fastrepl/char) | 7,716 | AI notepad for meetings | Minimal UI patterns |
| **qmd** (tobi/qmd) | 8,470 | CLI search engine for notes/docs | Local semantic search patterns |
| **PrivaNote_ts** | 0 | Local models (Gemma, Whisper.cpp) | Whisper.cpp integration |
| **talk-to-text-ai** | 0 | Silero VAD + Whisper | Voice activity detection (filter silence) |

---

## Competitive Positioning Matrix

| Feature | Fieldy | Plaud | Otter/Fireflies | Meetily (OSS) | **Pocket Consigliere** |
|---------|--------|-------|-----------------|---------------|------------------------|
| **Deployment** | Cloud + wearable | Cloud + wearable | Cloud SaaS | Local desktop | **Local desktop** |
| **Real-time suggestions** | No | No | No | No | **Yes (every 60s)** |
| **Client context system** | No | No | No | No | **Yes (profiles + history)** |
| **Vendor catalog** | No | No | No | No | **Yes (consultant-specific)** |
| **Speaker ID** | Yes | Yes | Yes | Yes | V2 |
| **Hands-free** | Yes (wearable) | Yes (wearable) | No | No | No |
| **Local processing** | No (cloud sync) | No (cloud sync) | No | **Yes** | **Yes** |
| **Custom LLM endpoint** | No | No | No | **Yes** | **Yes** |
| **Bot joins meeting** | No | No | **Yes** | No | No |
| **System audio capture** | No | No | No | **Yes** | **Yes** |
| **Price** | ~$200 + subscription | ~$200 + subscription | Free-$20/user/mo | Free (OSS) | **Free (internal tool)** |

---

## Our Competitive Advantages

### 1. Real-Time Strategic Suggestions (No One Does This)
**Gap in market:** All competitors do post-meeting summaries only.  
**Our advantage:** Real-time strategic suggestions every 60 seconds during the meeting.

**Why this matters:**
- Allows consultants to adapt strategy mid-call
- Surfaces questions/talking points while the conversation is still happening
- No competitor can easily add this (requires different UX + technical architecture)

---

### 2. Client Intelligence System (No One Does This)
**Gap in market:** No competitor has client profiles + vendor catalog + engagement history.  
**Our advantage:** Auto-prep mode with "what we discussed last time" + vendor overlap analysis.

**Why this matters:**
- Consultants need context across multiple engagements with same client
- Historical context informs better questions
- Vendor catalog enables cross-sell/gap analysis

---

### 3. Local-First, No Cloud Lock-In
**Advantage:** Privacy, data sovereignty, no recurring subscription.

**Competitors with this:**
- Meetily (OSS)

**Competitors without this:**
- All cloud SaaS products (Otter, Fireflies, Fathom)
- All hardware products (Fieldy, Plaud)

---

### 4. System Audio Capture (Invisible to Clients)
**Advantage:** No bot joins meeting, fully invisible.

**Competitors with this:**
- Meetily (OSS)
- Buzz (transcription only)

**Competitors without this:**
- Otter, Fireflies, Fathom (bot joins as participant)

---

## What We Can Learn While Keeping Our Requirements

### 1. Audio Capture (Steal from Meetily)
**Your risk:** Cross-platform audio is make-or-break.  
**Their solution:**
- Rust-based audio mixing (mic + system audio simultaneously)
- Intelligent ducking (prevent mic feedback when system audio plays)
- Clipping prevention
- Borrowed code from [screenpipe](https://github.com/mediar-ai/screenpipe)

**Action:**
- ✅ Fork Meetily's audio module or screenpipe's audio code
- ✅ Test on your target platform (macOS first)
- ✅ Build audio capture POC before building the rest of Pocket Consigliere

---

### 2. Local LLM (Steal from Meetily + Ollama Ecosystem)
**Your requirement:** Configurable STT/LLM endpoints (OpenAI-compatible).  
**Their solution:**
- Ollama for local inference (Llama, Mixtral, etc.)
- Fallback to Claude/Groq/OpenRouter for cloud
- Custom OpenAI endpoint support (exactly what you want)

**Action:**
- ✅ Add Ollama as a built-in option for LLM (not just "configure endpoint")
- ✅ Test latency: can Ollama on a MacBook Pro deliver <30s reasoning for your use case?

---

### 3. GPU Acceleration (Steal from Meetily)
**Your requirement:** Real-time transcription must be responsive.  
**Their solution:**
- Apple Silicon (Metal + CoreML)
- NVIDIA (CUDA)
- AMD/Intel (Vulkan)
- Auto-enabled at build time

**Action:**
- ✅ If using Whisper, compile with Metal support on macOS (4x faster than CPU)
- ✅ Test: does GPU-accelerated Whisper hit <5s latency for 60s audio chunks?

---

### 4. Privacy Messaging (Steal from Fieldy + Meetily)
**Fieldy:** "Privacy is a fundamental human right."  
**Meetily:** "Privacy-first AI meeting assistant. All data stays on your machine."  
**Plaud:** Leads with ISO 27001, GDPR, HIPAA compliance.

**Action:**
- ✅ Make privacy a selling point, not an afterthought.
- ✅ Landing page: "Your meetings. Your data. Your machine."
- ✅ Compliance-ready messaging even if you're not certified (V1 is 2-user internal tool, but position for future)

---

### 5. Auto-Reminder Extraction (Steal from Fieldy)
**Their feature:** AI generates reminders from your own words (no manual entry).  
**Your V1 requirement:** Action items extracted post-meeting.

**Action:**
- ✅ V1: Action items in post-meeting artifacts.
- ✅ V2: Auto-schedule reminders (integrate with calendar, or export to Todoist/Things)

---

### 6. Speaker Identification (Defer to V2, but learn from Meetily)
**Meetily:** Speaker diarization in V1 (OSS version).  
**Your requirement:** V2 (out of scope for V1).

**Action:**
- ✅ Don't build this in V1 (complexity explosion).
- ✅ But: study Meetily's diarization approach for when you're ready (V2).

---

### 7. Tauri vs Electron (Architecture Decision)
**Meetily uses Tauri:** Rust backend, smaller binaries, faster.  
**Your plan:** Electron.

**Tradeoff:**
| Criteria | Electron | Tauri |
|----------|----------|-------|
| Learning curve | Lower (Node.js/TypeScript) | Higher (Rust required) |
| Binary size | Larger (~150 MB) | Smaller (~10 MB) |
| Performance | Good | Better (native Rust) |
| Cross-platform audio | Mature Node modules | Rust crates (less mature) |
| Community/ecosystem | Huge | Growing |

**Recommendation:**
- ✅ Stick with Electron for V1 (faster to ship, mature ecosystem).
- ✅ Consider Tauri for V2 if performance becomes a bottleneck.

---

### 8. Failure Modes (Learn from Meetily's Architecture)
**Meetily's approach:**
- Graceful degradation: if GPU fails, fall back to CPU.
- Settings panel shows which features are enabled/disabled.
- Custom endpoint support means users can bypass failures (if Ollama breaks, use Claude).

**Action:**
- ✅ Build health check UI (show status of audio, STT, LLM, DB).
- ✅ Graceful degradation: if LLM fails, app still records + transcribes.

---

## Why Others Haven't Built Pocket Consigliere

### 1. Audio Capture is Hard
**Why most went with "bot joins meeting" model:**
- Avoids platform-specific audio APIs
- No permission dialogs
- Works across all meeting platforms without integration

**Why we're doing system audio anyway:**
- Invisible to clients (no bot participant)
- Works for in-person meetings (not just virtual)
- Privacy-first (no cloud server processing)

---

### 2. Small Market (Niche Use Case)
**Why competitors focus on broad market:**
- Otter/Fireflies: mass market (anyone in meetings)
- Fieldy/Plaud: mass market (anyone who wants notes)

**Why we're niche:**
- Target: high-end consultants only
- Need: client context + vendor catalog (specific to consulting)
- No network effects (local-first = no team collaboration)

---

### 3. Local LLM Inference is Challenging
**Why most use cloud APIs:**
- Most users don't have GPU hardware for real-time reasoning models
- Cloud APIs are easier to support
- Subscription model requires cloud dependency

**Why we're local-first:**
- Privacy (sensitive client conversations)
- Offline capability
- No recurring API costs

---

## Recommendations Based on Competitive Landscape

### **Steal Shamelessly from Meetily:**
1. ✅ Fork their audio capture code (it's MIT licensed)
2. ✅ Use their Whisper integration patterns
3. ✅ Study their cross-platform audio handling

### **Differentiate Aggressively:**
- **You:** Real-time strategic suggestions (not just transcription)
- **You:** Client knowledge base (not just meeting-by-meeting)
- **You:** Consultant-specific (vendor catalog, engagement history)

### **Validate Market Fit Early:**
**Risk:** Otter exists and is free for individuals. Why build your own?

**Answer:** 
- Privacy (local-first, no cloud)
- Real-time suggestions (not post-meeting only)
- Consultant-specific features (vendor catalog, client context)

**Test:** Use Otter for 2 weeks. Document every time you wish it had a feature Pocket Consigliere would have. If list is short, reconsider.

---

## Key Takeaways

### Your Moat:
1. **Real-time strategic suggestions** (no competitor does this)
2. **Client intelligence system** (no competitor has this)
3. **Consultant-specific features** (vendor catalog, engagement history)

### Technical Patterns to Steal:
1. **Audio capture** from Meetily (mic + system audio, ducking, clipping prevention)
2. **GPU acceleration** from Meetily (Metal/CUDA/Vulkan)
3. **Ollama integration** for local LLM inference
4. **Privacy messaging** from Fieldy/Meetily

### Risk Mitigation:
1. **Audio POC first** (validate before building everything else)
2. **Test Ollama latency** (<30s for 60s transcript + context?)
3. **Graceful degradation** (if LLM fails, app still records + transcribes)

---

## Related Documents

- [Detailed Requirements](pocket-consigliere-requirements.md)
- [Risk Considerations](risk-considerations.md)
- [Application Summary](pocket-consigliere-summary.md)
- [Implementation Plan](pocket-consigliere-implementation-plan.md)
