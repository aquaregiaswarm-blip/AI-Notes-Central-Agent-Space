# Pocket Consigliere — Risk Considerations & Technical Gotchas

**Version:** 0.1  
**Date:** 2026-02-15  
**Author:** Clarice (Technical Architecture Review)  
**Status:** Planning / Pre-Implementation  

---

## Purpose

This document captures critical risks, technical gotchas, and architectural considerations identified during requirements review. It applies Roy's decision framework to surface risks before implementation begins.

**Use this document to:**
- De-risk audio capture before building the rest of the app
- Define failure mode behavior
- Set performance targets
- Plan operational safeguards

---

## 🔴 Critical Gotchas (Will Bite You in V1)

### 1. Audio Capture is a Minefield

**Problem:** System audio + mic capture is OS-dependent hell.

**Platform-specific challenges:**
- **macOS:** Requires screen recording permissions (scary dialog), audio loopback (BlackHole/Loopback app), breaks across OS updates
- **Windows:** WASAPI works but driver issues are common, some audio devices don't expose loopback
- **Linux:** PulseAudio/PipeWire fragmentation

**What's missing from requirements:**
- Fallback strategy when audio capture fails (currently: app is useless)
- Permission request UX (macOS "screen recording" dialog will scare users)
- Audio device selection UI (which mic? which output?)
- Test plan for "what if user denies permissions?"

**Mitigation:**
- ✅ V1: Pick ONE platform (macOS), test ruthlessly, document setup
- ✅ Build "audio health check" on launch (can we capture? show status)
- ✅ Add manual "import audio file" fallback (record externally, drop in app)
- ✅ Week 1 spike: Build audio capture prototype BEFORE building everything else

**Decision:** This is a one-way door. Validate feasibility NOW before committing to Electron + native audio modules.

---

### 2. LLM Context Window Management

**Problem:** 60-second transcript chunks → cumulative context → token explosion within 30 minutes.

**Math:**
- ~150 words/min average speech
- 30-min meeting = 4,500 words ≈ 6,000 tokens (transcript alone)
- Add client profile + vendor catalog + prior meeting summaries: +2,000 tokens
- **Total: 8,000+ tokens** for a 30-min call, climbing to 20,000+ for 90-min calls

**What's missing from requirements:**
- Token budget enforcement (what happens when you hit model's context limit?)
- Sliding window strategy (keep last N minutes + summary of earlier conversation?)
- Cost estimation (if using paid APIs, this gets expensive fast)

**Mitigation:**
- ✅ Implement **sliding window**: last 10 minutes full transcript + summarized earlier context
- ✅ Add token counter UI (show remaining capacity in real-time)
- ✅ Test with 2-hour call simulation before launch
- ✅ Define max meeting length supported (recommend: 2 hours, graceful degradation beyond that)

---

### 3. Latency Perception = UX Killer

**Problem:** 60-second LLM analysis cycle feels slow. Users will think it's broken.

**Timing breakdown:**
- Audio capture: real-time
- STT: 5-15 seconds (depending on endpoint)
- LLM reasoning: 10-30 seconds (model-dependent)
- **Total lag: 15-45 seconds** between speech and suggestion

**What's missing from requirements:**
- UI feedback during processing ("Analyzing..." spinner)
- Explanation of why suggestions take time (set expectations)
- Test with real users: does 60-second cycle feel responsive enough?

**Mitigation:**
- ✅ Add progress indicator during LLM call
- ✅ Consider 30-second cycle for shorter meetings (<30 min), 90-second for longer
- ✅ Show "last analyzed" timestamp on each suggestion batch
- ✅ Performance target: LLM analysis must complete within 90 seconds or timeout

---

### 4. Notification Fatigue / Suggestion Overload

**Problem:** 5 suggestions every 60 seconds = 300 suggestions in a 1-hour meeting. Overwhelming.

**What's missing from requirements:**
- Severity/priority ranking (which suggestions matter most?)
- Suggestion expiration (old suggestions should fade/archive)
- User testing: how many suggestions per cycle is actually useful?

**Mitigation:**
- ✅ Start with 3 suggestions/cycle (not 5), make it configurable
- ✅ Add suggestion aging: dim/collapse after 5 minutes if not dismissed
- ✅ Track dismiss patterns: if user dismisses 80% of suggestions, reduce frequency
- ✅ V1.1: Suggestion quality analytics (dismiss rate, acknowledge rate)

---

### 5. Cross-Platform Audio is a One-Way Door

**Problem:** You're committing to Electron + native audio modules. This is hard to reverse.

**Roy's decision framework applied:**
1. **Does it work today?** No, you have to build it.
2. **Can the team operate it?** Unclear—debugging audio issues at 2 AM is nightmare fuel.
3. **Does it compound?** No—this is pure technical debt.
4. **Blast radius?** High—if audio breaks, app is useless.
5. **Is it reversible?** No—switching from Electron to native app later means full rewrite.

**What's missing from requirements:**
- Proof-of-concept for audio capture BEFORE building the rest of the app
- Windows/Linux feasibility testing (requirements say "investigate parity," but audio is make-or-break)

**Mitigation:**
- ✅ **Week 1 spike:** Build audio capture prototype. macOS + Windows. If Windows doesn't work reliably, drop it from V1 scope NOW.
- ✅ Document setup steps for users (install BlackHole, grant permissions, etc.)
- ✅ Decision point: If audio POC fails, pivot to "import recording" workflow instead of live capture

---

## 🟡 Important Gotchas (Manageable but Need Planning)

### 6. Vector DB Embedding Strategy

**Problem:** Every meeting generates transcript → needs embedding → slow + expensive.

**What's missing from requirements:**
- Embedding model choice (local vs API, speed vs quality tradeoff)
- Batch vs real-time embedding (embed during meeting or post-meeting?)
- Storage growth (1-hour meeting transcript = 9,000 words = 36 KB text, but embeddings = 1.5 MB)

**Mitigation:**
- ✅ Use local embedding model (all-MiniLM-L6-v2, fast + small)
- ✅ Embed post-meeting, not during (keeps UI responsive)
- ✅ Add "rebuild index" tool for when embeddings corrupt

---

### 7. "Human-in-the-Loop" Friction

**Problem:** Every organic update to client profile requires user approval. Good for safety, bad for UX.

**What's missing from requirements:**
- Approval queue UI (where do pending updates live?)
- Approval fatigue mitigation (batch approvals? auto-approve low-confidence changes?)
- What happens if user ignores approval queue for weeks?

**Mitigation:**
- ✅ Approval inbox (sidebar badge count)
- ✅ Confidence scores: auto-approve high-confidence, flag low-confidence
- ✅ Weekly digest: "You have 12 pending client updates"

---

### 8. Single-Executable Dream is Hard

**Problem:** "No separate services" is aspirational. Vector DB, SQLite, audio capture, STT, LLM—each has failure modes.

**What's missing from requirements:**
- Graceful degradation plan (if vector DB fails, can app still work?)
- Health check dashboard (which components are healthy?)
- Restart-without-quit for hung components

**Mitigation:**
- ✅ Build "system health" panel in settings (show status of each subsystem)
- ✅ If vector DB fails, app still works (just no semantic search)
- ✅ If STT fails, manual transcript entry fallback

---

### 9. Update Strategy

**Problem:** "Auto-update with human approval" is fine, but you're shipping an Electron app with embedded databases. Schema migrations during updates are risky.

**What's missing from requirements:**
- Rollback plan (if update breaks, can user downgrade?)
- Backup-before-update (automatic? manual?)
- Update testing strategy (how do you QA schema migrations?)

**Mitigation:**
- ✅ Auto-backup database before applying update
- ✅ Keep last 3 versions available (rollback via settings)
- ✅ Test migration path: v1.0 → v1.1 → v1.2 without data loss

---

### 10. Debugging in Production (It's Just You Two)

**Problem:** When it breaks during a live client call, you need fast diagnosis.

**What's missing from requirements:**
- Logging strategy (where do logs go? how verbose?)
- Error reporting (Sentry? Local logs only?)
- "Safe mode" launch (disable LLM, audio, etc. to isolate failures)

**Mitigation:**
- ✅ Structured logs to `~/.pocket-consigliere/logs/`
- ✅ Settings: enable verbose logging
- ✅ Command-line flag: `--safe-mode` (launches with all integrations disabled)

---

## ⚠️ What's Missing from Requirements

### 11. Failure Modes Not Documented

**Missing:**
- What happens when STT endpoint is down?
- What happens when LLM endpoint is slow (>60s response)?
- What happens when transcript exceeds context window?
- What happens when user loses internet mid-meeting (if using cloud STT/LLM)?

**Add to requirements:**
- Section: "Failure Mode Behavior"
- Document expected behavior for each failure scenario

---

### 12. Performance Targets Not Defined

**Missing:**
- How responsive must UI be? (<100ms? <1s?)
- Max acceptable LLM latency? (30s? 60s? Timeout after?)
- Max meeting length supported? (1 hour? 4 hours? All-day?)

**Add to requirements:**
- Non-functional requirements: "UI must respond to user input within 100ms"
- Non-functional requirements: "LLM analysis must complete within 90 seconds or timeout"
- Non-functional requirements: "Support meetings up to 2 hours (graceful degradation beyond)"

---

### 13. User Onboarding / First-Run Experience

**Missing:**
- How do users configure STT/LLM endpoints on first launch?
- What if they don't have API keys? (Provide defaults? Require setup?)
- Tutorial/walkthrough for first meeting?

**Add to requirements:**
- Story: "First-run wizard" (configure endpoints, test audio, create first client)

---

### 14. Client Profile Schema Evolution

**Missing:**
- What if client's tech stack changes over time? (Historical snapshots?)
- How do you track "what they had in Q1 vs Q3"?
- Discrepancy resolution: if two meetings contradict, how is conflict surfaced?

**Add to requirements:**
- Schema: `client_stack_history` table (timestamped snapshots)
- UI: Timeline view of client's tech evolution

---

### 15. Suggestion Quality Feedback Loop

**Missing:**
- How do you know if suggestions are useful?
- Track: dismiss rate, acknowledge rate, suggestion→action conversion
- Use feedback to improve prompts over time

**Add to requirements:**
- Analytics dashboard: "Suggestion quality over time"
- Export suggestion history for prompt tuning

---

### 16. Multi-Meeting Continuity

**Problem:** If you have 3 meetings with the same client in one week, does context carry forward?

**Missing:**
- "Current engagement" concept (group related meetings)
- Summary of "what we discussed last time" for auto-prep

**Add to requirements:**
- Engagement grouping (optional): link related meetings
- Auto-prep: "Last discussed 3 days ago: [summary]"

---

## 🟢 Recommendations (Prioritized)

### Must-Fix Before V1:

1. ✅ **Audio capture POC first** (validate feasibility before building everything else)
2. ✅ **Token budget enforcement** (sliding window + token counter UI)
3. ✅ **Latency feedback UI** (show "analyzing..." during LLM calls)
4. ✅ **Failure mode requirements** (document expected behavior when things break)
5. ✅ **Safe mode launch** (for debugging in production)

### Should-Fix in V1:

6. ✅ **Suggestion volume tuning** (start with 3/cycle, not 5)
7. ✅ **Approval queue UI** (for human-in-the-loop client updates)
8. ✅ **Health check dashboard** (show status of audio, STT, LLM, DB)
9. ✅ **Performance targets** (define acceptable latency/responsiveness)
10. ✅ **Backup-before-update** (automatic database backup)

### Can-Defer to V1.1:

11. Client stack history timeline
12. Suggestion quality analytics
13. Multi-meeting engagement grouping
14. Cross-platform audio (Windows/Linux)

---

## Roy's Decision Framework Applied

### Does it work today?
**No.** Significant build required. Audio capture is highest risk.

### Can the team operate it?
**Unclear.** Debugging audio issues, LLM timeouts, and database migrations at 2 AM is hard for a 2-person team.

**Mitigation:** Safe mode, health dashboard, structured logs.

### Does it compound?
**Yes**—if it works, every meeting makes the system smarter (more context, better suggestions).

**Risk:** If it's unreliable, you'll stop using it and the investment is wasted.

### What's the blast radius?
**High.** 
- If audio fails → app is useless
- If LLM is slow → UX is broken
- If database corrupts → you lose all history

**Mitigation:** Graceful degradation, backups, rollback plan.

### Is it reversible?
**No.** Electron + native audio modules = one-way door. Hard to switch later.

**Mitigation:** Build audio POC first to validate feasibility.

---

## Bottom Line

This is **ambitious but achievable** IF you:

1. ✅ De-risk audio capture early (Week 1 spike)
2. ✅ Build in operational safeguards (health checks, safe mode, backups)
3. ✅ Define failure modes and performance targets before implementation
4. ✅ Test with 2-hour call simulation before calling it "done"

**The requirements are solid, but you need to:**
- Flesh out failure mode behavior
- Set performance targets
- Build audio POC before committing to full implementation

**Recommendation:** Treat audio capture as a go/no-go decision point. If Week 1 audio POC doesn't work reliably, pivot to "import recording" workflow instead of live capture.

---

## Related Documents

- [Detailed Requirements](pocket-consigliere-requirements.md)
- [Application Summary](pocket-consigliere-summary.md)
- [Implementation Plan](pocket-consigliere-implementation-plan.md)
- [Ideation Transcript](transcripts/2026-02-14-pocket-consigliere-ideation.md)
