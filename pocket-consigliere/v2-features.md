# Pocket Consigliere — V2 Feature Backlog

**Version:** 0.1  
**Date:** 2026-02-15  
**Status:** Future Planning  

---

## Purpose

This document captures features deferred from V1 and new ideas for V2 and beyond. Features are categorized by priority and complexity.

---

## High Priority (V2 Focus)

### 1. Meeting Quality Checklist System

**Proposed by:** Jonathan Gough (2026-02-15)

**Problem:**
Consultants and sales reps often miss critical qualification questions or next-step discussions during live calls. Post-meeting, they realize they forgot to ask about budget, confirm authority, or set a follow-up timeline.

**Solution:**
Pre-defined meeting checklists based on meeting type (discovery, qualification, follow-up, close) that ensure all critical topics are covered. The system tracks what's been discussed and prompts the user if key items are missing.

**Key Features:**

#### BANT Qualification Tracker
- **Budget:** "Have we identified budget? Is there allocated funding?"
- **Authority:** "Who is the decision-maker? Do we need additional stakeholders?"
- **Need:** "Have we confirmed the business problem? What's the impact of not solving it?"
- **Timing:** "What's the timeline for decision? Any forcing functions?"

**Visual UI:**
- Checklist sidebar (✓ covered, ⚠ partially covered, ✗ not discussed)
- Real-time updates as LLM detects topics in transcript
- Color-coded: green (done), yellow (partial), red (missing)

#### Standard Sales/Consulting Must-Dos

**Discovery meetings:**
- Confirm business problem
- Understand current state
- Identify stakeholders
- Set next steps

**Qualification meetings:**
- BANT qualification
- Technical requirements
- Success criteria
- Decision process

**Follow-up meetings:**
- Review action items from last meeting
- Address open questions
- Confirm timeline
- Identify blockers

**Close meetings:**
- Final objections surfaced
- Contract terms agreed
- Next steps clear
- Signatures/approvals

#### Customizable Checklists
- Define custom checklists per client or engagement type
- Example: "Enterprise deals need security review confirmation"
- Example: "Public sector deals need procurement timeline"

#### Post-Meeting Completeness Score
- "You covered 7/10 critical topics"
- "Missing: Budget confirmation, decision timeline"
- Prompt for follow-up email to address gaps

#### LLM Auto-Detection
- LLM analyzes transcript for checklist items
- Flags if critical topics not discussed
- Suggests questions to ask before meeting ends

#### Historical Tracking
- Did we get BANT in first call? Second call? Third call?
- Track which meetings covered which topics
- Identify patterns: "We always forget to ask about budget in discovery calls"

**Use Cases:**

**Use Case 1: Discovery Call**
- User starts meeting, selects "Discovery" checklist
- LLM monitors conversation
- After 20 minutes, checklist shows:
  - ✓ Business problem confirmed
  - ✓ Current state understood
  - ⚠ Stakeholders partially identified (only mentioned CTO, not VP Eng)
  - ✗ Next steps not set
- App suggests: "Ask: Who else should be involved in this decision?"
- App suggests: "Before ending call, confirm next steps and timeline"

**Use Case 2: Qualification Call**
- User starts meeting, selects "Qualification" checklist
- BANT tracker shows:
  - ✓ Budget: $200K allocated in Q2
  - ✗ Authority: Decision-maker not confirmed
  - ✓ Need: Business impact quantified
  - ⚠ Timing: "By end of quarter" but no forcing function
- App suggests: "Ask: Who has final approval authority?"
- App suggests: "Ask: What happens if we don't solve this by Q2?"

**Use Case 3: Post-Meeting Review**
- Meeting ends, completeness score: 6/8 topics covered
- Missing topics: Budget discussion, follow-up timeline
- App suggests follow-up email template:
  > "Quick follow-up: Can you confirm the budget allocated for this project? Also, when works best for our next check-in?"

**Technical Approach:**

- **Data model:** `meeting_checklist_templates` table (checklist name, items, order)
- **Data model:** `meeting_checklist_progress` table (meeting_id, checklist_item_id, status, detected_at)
- **LLM prompt enhancement:** Include checklist items in context, ask LLM to flag coverage
- **UI component:** Collapsible sidebar with checklist items
- **Post-meeting artifact:** Checklist completion report

**Complexity:**
- 🟡 Medium (new UI component, LLM prompt enhancement, checklist data model)

**Dependencies:**
- V1 baseline (transcript analysis, LLM integration)
- Post-meeting artifacts system

**Deferred because:**
- V1 focuses on core functionality (audio, transcription, real-time suggestions)
- Checklist system requires user testing to refine templates
- Not essential for MVP (nice-to-have, not must-have)

---

## Medium Priority (V2 or V3)

### 2. Speaker Diarization

**Problem:** Multi-person meetings lose context when transcript doesn't distinguish speakers.

**Solution:** Identify and label speakers in transcript (Speaker 1, Speaker 2, etc.). Optional: map to known contacts (client names).

**Complexity:** 🔴 Hard (requires advanced audio processing)

---

### 3. Conversation Pattern Detection

**Problem:** Consultants want to know if client is engaged, confused, or showing buying signals.

**Solution:** LLM analyzes sentiment, engagement patterns, and behavioral cues.

**Features:**
- Detect buying signals ("What's the pricing?" = high intent)
- Detect objections/concerns ("We're worried about..." = flag for follow-up)
- Detect confusion ("I'm not sure I understand..." = explain better)

**Complexity:** 🟡 Medium (LLM prompt enhancement)

---

### 4. External Internet Research

**Problem:** During a call, client mentions a competitor or technology you're unfamiliar with.

**Solution:** Real-time web search to surface context mid-meeting.

**Features:**
- LLM detects when external info would help ("They mentioned Datadog")
- Quick search result in suggestion panel
- Optional: auto-summarize competitor website

**Complexity:** 🟡 Medium (web search API integration, relevance filtering)

---

### 5. Multi-Meeting Engagement Grouping

**Problem:** Three meetings with same client in one week should be treated as one "engagement."

**Solution:** Group related meetings, surface "what we discussed last time" automatically.

**Features:**
- "Current engagement" concept (link related meetings)
- Auto-prep: "Last discussed 3 days ago: [summary]"
- Continuity tracking: "In our first call, you said X. Has that changed?"

**Complexity:** 🟡 Medium (data model change, auto-prep logic)

---

### 6. Client Tech Stack Timeline

**Problem:** Client's tech changes over time (Q1 vs Q3). No way to track evolution.

**Solution:** Historical snapshots of client tech stack, timeline view.

**Features:**
- `client_stack_history` table (timestamped snapshots)
- UI: Timeline view of client's tech evolution
- Detect when tech stack changes between meetings

**Complexity:** 🟢 Easy (schema addition, timeline UI component)

---

### 7. Suggestion Quality Analytics

**Problem:** How do you know if suggestions are useful?

**Solution:** Track dismiss rate, acknowledge rate, suggestion→action conversion.

**Features:**
- Dashboard: "Suggestion quality over time"
- Export suggestion history for prompt tuning
- Flag low-performing suggestion types

**Complexity:** 🟢 Easy (analytics dashboard)

---

## Low Priority (Future)

### 8. Native Meeting Platform Integrations

**Problem:** Some users want bot-joins-meeting model (like Otter).

**Solution:** Integrate with Teams, Zoom, WebEx APIs as native participant.

**Complexity:** 🔴 Hard (platform-specific SDKs, OAuth flows, compliance)

**Deferred because:** System audio capture is our differentiation (invisible to clients).

---

### 9. Mobile App

**Problem:** Consultants want meeting prep on mobile (review client profile before call).

**Solution:** Companion mobile app (iOS/Android) for meeting prep and post-meeting review.

**Complexity:** 🔴 Hard (new platform, sync architecture)

---

### 10. Multi-User / Team Collaboration

**Problem:** Consulting teams want shared client profiles and meeting history.

**Solution:** Multi-tenant architecture with team workspaces.

**Complexity:** ⛰️ Very Hard (authentication, authorization, data isolation, sync)

**Deferred because:** V1 is 2-user internal tool. Team features are product pivot.

---

### 11. Advanced Security/Compliance

**Problem:** Enterprise customers need SOC2, GDPR, HIPAA compliance.

**Solution:** Encryption at rest, audit logs, SSO, role-based access.

**Complexity:** 🔴 Hard (security audit, compliance certification)

---

### 12. Automated Email Sending

**Problem:** Users want post-meeting summaries auto-sent to client.

**Solution:** Email integration (Gmail, Outlook) with template-based sending.

**Complexity:** 🟡 Medium (email API integration, template engine)

**Deferred because:** V1 focuses on copy/paste workflow (lower risk, simpler UX).

---

## Priority Ranking (Next Up)

**For V2 planning, prioritize:**

1. ✅ **Meeting Quality Checklist System** (high value, medium complexity, consultant-specific)
2. ✅ **Conversation Pattern Detection** (enhances real-time suggestions, medium complexity)
3. ✅ **Multi-Meeting Engagement Grouping** (solves continuity problem, medium complexity)
4. ✅ **Client Tech Stack Timeline** (easy win, useful for long-term clients)

**Defer to V3+:**
- Speaker diarization (hard, less critical)
- External research (medium, less critical)
- Native platform integrations (hard, conflicts with our positioning)
- Mobile app (hard, separate product effort)

---

## Related Documents

- [Detailed Requirements (V1)](pocket-consigliere-requirements.md)
- [Risk Considerations](risk-considerations.md)
- [Market Comparison](market-compare-and-contrast.md)
- [Ideas Backlog](../ideas/backlog.md)
