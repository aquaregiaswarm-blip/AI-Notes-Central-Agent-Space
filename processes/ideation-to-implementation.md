# Ideation to Implementation — Planning Methodology

**Type:** Process Template  
**Author:** Captured from Jonathan Gough ideation session (2026-02-14)  
**Use Case:** Scoping new applications from idea to actionable implementation plan

---

## Overview

This methodology transforms a raw application idea into a complete implementation plan through structured Q&A, iterative refinement, and progressive detail expansion. It was developed during the Pocket Consigliere ideation session and captures Jonathan Gough's approach to product scoping.

**Key Principle:** "We're not building anything yet. Your job is to ask us questions about the application — how it would work, what functionality we need, what we don't need — so we can fully scope out if we want to do this and how we would do it."

---

## Phase 1: Initial Concept Capture

### Objective
Understand the core idea without judgment or premature solution design.

### What the Facilitator Does
1. **Listen first** — let the product owner describe the idea in their own words
2. **Capture the vision** — what problem does this solve?
3. **Note constraints early** — platform, users, timeline
4. **Identify the success criteria** — what does "done" look like?

### Questions to Ask
- Who is this for? (target users, personas)
- What problem are they trying to solve?
- What does success look like from their perspective?
- Are there existing solutions they've tried? What's missing?

### Example from Pocket Consigliere
- **Idea:** Real-time AI meeting copilot for consultants
- **Users:** Jonathan & Roy (v1), high-end consultants talking to senior client stakeholders
- **Problem:** Need strategic guidance during client meetings without visible prompts or delays
- **Success:** AI that feels like a brilliant, silent partner who's done the research

---

## Phase 2: Structured Clarification (10+ Questions)

### Objective
Flesh out functionality, UI/UX, data model, and scope through deliberate Q&A.

### The "10 Questions" Approach
**Rule:** Ask **no less than 10 clarifying questions**, but **one at a time** — not all at once.

**Why one-by-one?**
- Allows product owner to think through each decision independently
- Prevents overwhelm
- Uncovers dependencies and conflicts early
- Gives facilitator time to synthesize before the next question

### Question Categories

#### 1. **Functional Requirements**
- How does the core workflow work step-by-step?
- What inputs does the system receive?
- What outputs does it produce?
- What happens when [edge case] occurs?

#### 2. **User Experience**
- How does the user consume the output? (UI, notifications, export)
- What actions can the user take in response?
- How intrusive should the system be during use?
- What should happen if the user ignores a suggestion?

#### 3. **Data & Storage**
- What data needs to persist?
- How should historical data be structured?
- What needs to be searchable?
- What's the tolerance for data loss?

#### 4. **Context & Intelligence**
- What context does the AI need to be useful?
- Where does that context come from? (user input, historical data, external sources)
- How does the system avoid redundancy or irrelevance?
- What's the update frequency for contextual data?

#### 5. **Integrations & Dependencies**
- What external systems does this connect to?
- Are those integrations required for v1 or nice-to-have?
- What happens if an external service is unavailable?

#### 6. **Scope Boundaries**
- What's explicitly out of scope for v1?
- What features are "upgrade path" (v2, v3)?
- What's the "insurmountable hill" (don't even try)?

#### 7. **Technical Constraints**
- What platforms must this run on?
- Are there performance requirements? (latency, throughput)
- What's the deployment model? (cloud, local, hybrid)
- Are there security or compliance constraints?

#### 8. **Simplicity vs. Fragility**
- How many moving pieces can the system tolerate?
- What's the "easy button" version?
- What's the complexity ceiling before it becomes unmaintainable?

### Example Questions from Pocket Consigliere

1. Who is the target user? → Jonathan & Roy (v1 only)
2. How should the output be consumed during a live call? → Overlay/sidebar
3. What audio needs to be captured? → System audio + microphone
4. Is speaker identification required? → No for v1, diarization is v2
5. What context does the AI need? → Client profile, vendor catalog, meeting history
6. How should suggestions be triaged? → ✓ (acknowledge) or ✗ (dismiss)
7. What happens after the meeting? → Generate transcript, summary, action items
8. How is data stored? → SQLite + vector DB + flat markdown files
9. What's out of scope for v1? → External research, native meeting integration, multi-user
10. What's the deployment model? → Fully local Electron app, single executable

---

## Phase 3: Iterative Synthesis

### Objective
After every 2-3 questions, **summarize what you've captured so far** and confirm understanding.

### Synthesis Pattern

```
What I've captured so far:
- [Core concept]
- [Key decisions]
- [Constraints]
- [Open questions]

Does this match your vision? Anything I've misunderstood?
```

### Why This Matters
- Keeps product owner and facilitator aligned
- Surfaces misunderstandings early
- Demonstrates active listening
- Creates checkpoints for scope evolution

### Example from Pocket Consigliere
After question 5, Clarice synthesized:
> **Concept:** Real-time meeting copilot (Electron on Mac)  
> **Flow:** Audio → STT → LLM (every ~60s) → suggestions  
> **Context:** Call itself, client, org capabilities  
> **Output:** Questions, notes, research (non-redundant)  
> **Users (v1):** Jonathan & Roy  
> **Audio:** System + mic, no speaker ID initially  
> **Meetings:** Primarily Microsoft Teams

This confirmed alignment before moving to UI/UX questions.

---

## Phase 4: Document Generation

### Objective
Transform the Q&A into structured artifacts that enable implementation.

### Artifact Hierarchy
1. **Requirements Document** — what the system must do (functional + non-functional)
2. **Application Summary** — one-pager for stakeholders (elevator pitch + feature list)
3. **Complexity Assessment** — easy → hard → insurmountable (helps prioritize)
4. **Epics & Stories** — implementation chunks with acceptance criteria
5. **Implementation Plan** — tasks, subtasks, tests, dependencies
6. **Process Capture** — the methodology itself (meta-documentation)

### Requirements Document Structure
```
1. Overview (what + why)
2. Target Users (personas, constraints)
3. Platform Requirements (OS, deployment, self-contained?)
4. Core Functional Requirements (categorized by domain)
5. Data Storage Requirements
6. Configuration & Settings
7. Non-Functional Requirements (simplicity, resilience, privacy)
8. Explicitly Out of Scope (v1)
9. Complexity Assessment (easy → insurmountable)
10. Recommended Tech Stack (v1)
```

### Epics & Stories Structure
```
Epic: High-level capability (e.g., "Audio Capture & STT")
  └─ Story: User-facing feature (e.g., "As a user, I want...")
      ├─ Acceptance Criteria
      ├─ Dependencies
      └─ Priority (v1, v2, future)
```

### Implementation Plan Structure
```
Story:
  ├─ Tasks (high-level work items)
  ├─ Subtasks (granular steps)
  ├─ Test Plans:
  │   ├─ Unit Tests
  │   ├─ Functional Tests
  │   ├─ Integration Tests
  │   └─ E2E Tests
  ├─ Dependencies & Blockers
  └─ Acceptance Criteria
```

---

## Phase 5: Validation & Refinement

### Objective
Present the artifacts back to the product owner for review and iteration.

### Validation Checklist
- [ ] Does the requirements doc match the original vision?
- [ ] Are all edge cases addressed?
- [ ] Is the complexity assessment realistic?
- [ ] Are the epics/stories actionable?
- [ ] Is the scope tight enough for v1?
- [ ] Are dependencies and blockers identified?

### Refinement Loop
1. **Present draft** → get feedback
2. **Identify gaps** → ask follow-up questions
3. **Update artifacts** → incorporate changes
4. **Re-present** → confirm alignment
5. **Lock scope** → move to implementation

### Example from Pocket Consigliere
After the initial epic/story draft, Jonathan requested:
> "For each epic, create tasks and subtasks for every story. Outline all necessary unit tests, functional tests, integration tests, and end-to-end tests. Outline dependencies/blockers/etc."

This expanded the plan from high-level stories to granular, shippable work.

---

## Phase 6: Methodology Capture (Meta)

### Objective
Document the process itself so it can be reused.

### What to Capture
- The question types that worked
- The synthesis checkpoints that kept alignment
- The artifact structure that enabled clarity
- The refinement loop that caught gaps

### Why This Matters
- Turns a good session into a repeatable process
- Enables others to facilitate similar sessions
- Creates a feedback loop for process improvement
- Builds organizational memory

---

## Phase 7: Generate AI Assistant Instructions (NEW)

### Objective
Create setup instructions for AI coding assistants (Claude Code, Cursor, GitHub Copilot) so they can implement the project with full context.

### What to Generate
A `CLAUDE.md` (or `.ai/INSTRUCTIONS.md`) file that includes:
1. **Project overview** (2-3 paragraphs: what, why, who, success criteria)
2. **Architecture summary** (key decisions + rationale)
3. **Tech stack** (with "why chosen" for each major component)
4. **Epic sequence** (build order with dependencies)
5. **Critical gotchas** (things that will break if not handled correctly)
6. **Test strategy** (when to test, performance targets)
7. **File structure** (where to find planning docs)
8. **Development workflow** (how to get started)
9. **Clarifying questions** (common ambiguities resolved upfront)

### Template
Use `templates/CLAUDE.md.template` as starting point.

### When to Generate
**After** all planning artifacts are complete:
- Requirements ✅
- Charter ✅
- Architecture ✅
- Epics & Stories ✅
- Implementation Plan ✅

### Why This Matters
- **Maximum context available:** Planning session has full context fresh in memory
- **Standardized handoff:** AI assistants start with complete understanding
- **No cold start:** Developer clones repo, AI reads CLAUDE.md, ready to build
- **Alignment:** AI instructions match planning artifacts (no drift)
- **Iterative improvement:** Template evolves as we learn what works

### Example Section (Epic Sequence)

```markdown
## Epic Sequence (Build Order)

**IMPORTANT:** Build in this order. Dependencies are critical.

| Epic | Priority | Why First | Blockers |
|------|----------|-----------|----------|
| Epic 0: Week 1 POC | **CRITICAL** | Go/no-go validation of audio+STT+LLM chain | None |
| Epic 1: Foundation | Must-have | Electron app scaffold, all other epics depend on this | Epic 0 |
| Epic 2: Audio Capture | Must-have | Rust addon must work before STT | Epic 1 |
| Epic 3: STT | Must-have | Needs audio capture functional | Epic 2 |

**After each Epic:**
- ✅ Run tests for that Epic
- ✅ Commit and push (checkpoint)
- ✅ Ask: "Epic [N] complete. Ready for Epic [N+1]?"
```

### Integration with Planning Session

**At end of planning session, facilitator asks:**
> "Let's generate AI assistant instructions (CLAUDE.md) so this project can be implemented by Claude Code or Cursor."

**Then:**
1. Use template as starting point
2. Fill in sections based on planning artifacts
3. Highlight critical gotchas from risk analysis
4. Define Epic sequence with dependencies
5. Save as `CLAUDE.md` or `.ai/INSTRUCTIONS.md`
6. Commit with planning artifacts

### Benefits Over "Claude /init"

**Traditional approach:** Developer runs `/init`, Claude inspects repo, generates understanding

**Problem:**
- Claude's understanding might diverge from planning intent
- No standardized format (each Claude.md is different)
- Misses context from planning session (verbal decisions not in docs)

**New approach:** Generate instructions during planning

**Benefits:**
- ✅ Planning context captured while fresh
- ✅ Standardized format (template-based)
- ✅ Verbal decisions documented
- ✅ AI starts with correct understanding (no drift)
- ✅ Template improves over time (organizational learning)

### What Goes in CLAUDE.md vs Other Docs

**CLAUDE.md:** AI assistant-specific guidance
- Build order (Epic sequence)
- Common pitfalls (gotchas)
- Workflow (how to commit, when to test)
- Clarifications (resolve ambiguities)

**Other docs:** Detailed specifications
- Requirements: What to build
- Architecture: How it's designed
- Implementation Plan: Detailed tasks

**Think of CLAUDE.md as the "getting started guide for AI assistants"** — it references detailed docs but doesn't replace them.

---

## Key Principles (Distilled)

### 1. **"No Less Than 10 Questions, One at a Time"**
Forces thorough exploration without overwhelming the product owner.

### 2. **"We're Not Building Yet"**
Keeps the focus on understanding, not solutioning.

### 3. **"If We Have Too Many Moving Pieces, It'll Break"**
Simplicity is a first-class requirement, not an afterthought.

### 4. **"Ask Roy and I" (Not "Users")**
Specificity > generalization. Design for real people, not personas.

### 5. **Iterative Synthesis > One Big Dump**
Confirm understanding every 2-3 questions. Alignment is continuous, not final.

### 6. **Capture Everything, Filter Later**
Don't self-censor during ideation. Mark things as "v2" or "future" instead.

### 7. **Complexity Assessment is a Feature**
Explicitly calling out "hard" and "insurmountable" saves months of wasted effort.

### 8. **Document the Process Itself**
The meta-artifact (this document) is as valuable as the requirements doc.

---

## Recommended Question Flow

### Opening (Questions 1-3)
- Who is this for?
- What problem does it solve?
- What does success look like?

### Core Functionality (Questions 4-7)
- How does the workflow work?
- What are the inputs and outputs?
- What happens in edge cases?
- What integrations are required?

### User Experience (Questions 8-10)
- How does the user interact with the output?
- What level of intrusiveness is acceptable?
- What controls does the user need?

### Constraints & Scope (Questions 11-13)
- What's out of scope for v1?
- What's the deployment model?
- What's the "easy button" version?

### Data & Context (Questions 14-16)
- What data needs to persist?
- Where does contextual intelligence come from?
- How does the system avoid redundancy?

### Technical Details (Questions 17-20)
- What platform(s) must this run on?
- Are there performance requirements?
- What's the complexity ceiling?
- What's the recommended tech stack?

---

## Anti-Patterns to Avoid

### ❌ Asking All Questions at Once
Overwhelms the product owner, leads to shallow answers.

### ❌ Solution Before Scope
"We could use Kubernetes!" — but do we even need multiple services?

### ❌ Skipping Synthesis
Long Q&A without checkpoints → misalignment discovered too late.

### ❌ Ignoring Complexity Assessment
Assuming everything is "medium" difficulty → surprises during implementation.

### ❌ No Out-of-Scope List
Every feature seems important → scope creep → nothing ships.

### ❌ Forgetting the Meta
Great session, no documentation → can't replicate it next time.

---

## Success Metrics

A successful ideation session produces:

1. **Requirements doc** that could be handed to any competent dev team
2. **Epics & stories** that are immediately actionable
3. **Complexity assessment** that prevents wasted effort on insurmountable problems
4. **Alignment** between product owner and facilitator (no surprises in review)
5. **Confidence** that v1 scope is achievable
6. **Process doc** (this file) that enables the next session

---

## Tooling Recommendations

- **Note-taking:** Live markdown doc, shared screen
- **Synthesis:** After every 2-3 questions, paste summary back into chat
- **Artifacts:** Generate during session, review after
- **Validation:** Present drafts immediately, iterate quickly
- **Meta-capture:** Document the process while it's fresh

---

## Example Session Timeline

| Time | Phase | Activity |
|------|-------|----------|
| 0-10 min | Concept Capture | Product owner describes idea |
| 10-40 min | Structured Q&A | 10+ questions, one at a time, with synthesis checkpoints |
| 40-60 min | Requirements Draft | Facilitator writes first draft |
| 60-75 min | Review & Refine | Product owner reviews, provides feedback |
| 75-120 min | Epics & Stories | Facilitator expands into actionable chunks |
| 120-150 min | Implementation Plan | Full task breakdown with tests & dependencies |
| 150-165 min | Process Capture | Document the methodology itself |

---

## Adaptation Guide

### For Different Domains

**Hardware/IoT Products:**
- Add questions about physical constraints, power, connectivity
- Complexity assessment includes supply chain, certifications

**API/Backend Services:**
- Focus on contracts, SLAs, error handling
- Complexity assessment includes scalability, observability

**Mobile Apps:**
- Platform-specific constraints (App Store, permissions)
- Offline-first considerations

**Internal Tools:**
- Deployment friction is higher priority than polish
- "Good enough" > "pixel perfect"

### For Different Team Sizes

**Solo/Pair (2 people):**
- Shorter session (60-90 min)
- Skip formal epics, jump to task list
- Complexity assessment is optional

**Small Team (3-5 people):**
- Standard process (as documented)
- Epics & stories provide coordination structure

**Larger Team (6+ people):**
- Add acceptance test scenarios
- Include team capacity planning
- Delegate epic ownership during session

---

## Conclusion

This methodology transforms "I have an idea" into "here's exactly what to build and in what order."

The magic isn't in the artifacts — it's in the **deliberate, iterative questioning** that forces clarity before commitment.

**Use this process when:**
- Starting a new project from scratch
- Scoping a major feature addition
- Rescuing a project that's lost direction
- Teaching junior PMs how to facilitate ideation

**Don't use this process when:**
- The problem is well-understood (just build it)
- Time pressure demands immediate action (triage first, scope later)
- Political dynamics require a different facilitation style

---

**Final Note:**  
The best ideation sessions feel like conversations, not interrogations. The questions are a scaffold, not a script. Adapt, improvise, and always ask: "What else do I need to know?"

---

*Captured from Pocket Consigliere ideation session (2026-02-14)*  
*Participants: Jonathan Gough, Roy Bales, Clarice*  
*See: `pocket-consigliere/transcripts/2026-02-14-pocket-consigliere-ideation.md`*
