# Pocket Consigliere: Claude Code Implementation Workflow

**Date:** 2026-02-15  
**Developer:** Jonathan Gough  
**Tool:** Claude Code (Claude Desktop with MCP)  
**Approach:** Plan mode implementation with structured checkpoints

---

## Overview

This document captures the workflow Jonathan used to implement Pocket Consigliere using Claude Code in plan mode, going from planning artifacts to working code.

---

## Prerequisites

### 1. Planning Artifacts Complete
- ✅ Requirements document
- ✅ Epics & Stories (23 stories total)
- ✅ Implementation plan (detailed task breakdown)
- ✅ Architecture decision documented
- ✅ Risk analysis completed

### 2. Development Environment Setup
- **Tool:** Claude Code (Claude Desktop)
- **Mode:** Plan mode (structured, checkpoint-based implementation)
- **MCP Configuration:** Standard agents and MCP settings file added
- **Repository:** Local clone of project repo

---

## Workflow Steps

### Step 1: Configure Claude Code Environment

**Action:** Add standard agents and MCP settings file

**Purpose:**
- Enable Claude Code to interact with local filesystem
- Configure MCP (Model Context Protocol) for tool access
- Set up agents for structured implementation

**Files added:**
- `.claude/agents.json` (or similar)
- `.claude/mcp-settings.json` (or similar)

---

### Step 2: Initialize Claude with Repository Context

**Command:** `/init`

**What happened:**
1. Claude Code inspected the entire repository
2. Read planning documents (requirements, epics, implementation plan)
3. Understood project structure, architecture decisions, tech stack
4. **Generated:** `Claude.md` file

**Claude.md contents:**
- Repository structure overview
- Key architecture decisions
- Tech stack summary
- Implementation approach
- Notes on epic sequencing

**Action:** Committed `Claude.md` to repository

**Why this matters:**
- Creates a shared understanding between developer and AI
- Documents Claude's interpretation of the project
- Serves as reference for implementation decisions
- Checkpoint: Confirms Claude "gets it" before code is written

---

### Step 3: Enter Plan Mode and Begin Implementation

**Command:** "Start building"

**What happened:**
1. **Claude re-examined codebase** (to confirm latest state)
2. **Asked clarifying questions** (2-3 questions to resolve ambiguities)
3. **Started building** (Epic by Epic, following implementation plan)

**Clarifying questions Claude asked:**
- (Example: "Should I start with Epic 0 (Week 1 POC) or Epic 1 (Foundation)?")
- (Example: "Do you want me to use TypeScript strict mode?")
- (Example: "Should I bundle the Rust addon now or defer to later epic?")

**Implementation approach:**
- One Epic at a time (following the defined sequence)
- Tasks → Subtasks → Tests (per implementation plan)
- Checkpoint after each Epic (commit + push)

---

### Step 4: Iterative Epic Implementation

**Process:**

For each Epic:
1. **Claude reads Epic goals and stories** (from implementation plan)
2. **Implements tasks** (code generation, file creation, configuration)
3. **Writes tests** (unit, functional, integration as defined)
4. **Runs basic validation** (syntax check, build succeeds)
5. **Commits and pushes** (checkpoint after Epic completion)

**Commit strategy:**
- One commit per Epic (not per subtask)
- Commit message follows template: `"Epic X: [Epic Name] - [Brief summary]"`
- Push immediately after commit (ensures work is backed up)

**Example commit messages:**
```
Epic 1: Foundation - Electron app scaffold, SQLite, Vector DB, IPC
Epic 2: Audio Capture (Rust Addon) - Fork Meetily audio, napi-rs bindings
Epic 3: Speech-to-Text - Whisper.cpp integration, GPU acceleration
```

**Why commit after every Epic:**
- ✅ Creates checkpoint if anything breaks
- ✅ Clear progress tracking
- ✅ Easier to review changes (one Epic at a time)
- ✅ Reduces risk of losing work

---

### Step 5: Post-Implementation Testing

**After all Epics completed:**

**Developer:** "Did you run all the tests?"

**Claude:** "No, I have not run the full test suite yet. Let me do that now."

**What happened:**
1. Claude ran full test suite (unit, functional, integration, E2E)
2. **Found 1 error** (specific test failure)
3. **Diagnosed issue** (explained what was wrong)
4. **Fixed error** (code change + test update)
5. **Re-ran tests** (confirmed all passing)
6. **Committed fix** (final checkpoint)

**Example error:**
- Test: "Audio capture should merge mic + system audio"
- Issue: Audio streams not synchronized (timing issue)
- Fix: Added buffer alignment logic
- Result: Test passing

**Why this matters:**
- Tests were written during Epic implementation but not fully executed
- Running full suite at the end catches integration issues
- Claude can fix errors autonomously when given context

---

## Key Insights

### 1. **Plan Mode is Critical**
- Without plan mode, Claude might jump around between tasks
- Plan mode enforces Epic sequencing (dependencies respected)
- Checkpoints (commits) prevent backtracking

### 2. **/init Creates Shared Understanding**
- Claude.md documents Claude's interpretation
- Developer can review and correct before code is written
- Avoids misalignment ("I thought you meant X, not Y")

### 3. **Clarifying Questions are a Feature, Not a Bug**
- Claude asks 2-3 questions before starting (not 20)
- Questions surface ambiguities in planning docs
- Better to answer questions upfront than fix assumptions later

### 4. **Commit After Every Epic**
- Creates clear rollback points
- Easier code review (one Epic at a time)
- Progress visible in Git history

### 5. **Test Execution is Separate from Test Writing**
- Claude writes tests during implementation (per Epic)
- But doesn't run full suite until asked
- Final test pass catches integration issues

---

## Workflow Comparison: Human vs Claude Code

| Step | Human Developer | Claude Code (Plan Mode) |
|------|-----------------|--------------------------|
| **Read requirements** | Skim docs, ask questions | `/init` (full repo inspection) |
| **Understand architecture** | Draw diagrams, take notes | Generate `Claude.md` (shared context) |
| **Ask clarifying questions** | Slack/email back-and-forth | 2-3 questions before starting |
| **Implement Epic** | Hours/days per Epic | Minutes/hours per Epic |
| **Write tests** | Often deferred ("I'll write tests later") | Written alongside code (per plan) |
| **Run tests** | Periodically (when remembered) | On request (or CI/CD) |
| **Commit cadence** | Variable (per developer habit) | Consistent (after every Epic) |
| **Context switching** | High (distractions, meetings) | Low (focused on one Epic at a time) |

---

## Lessons Learned

### ✅ What Worked Well

1. **Planning artifacts were comprehensive**
   - Claude had everything needed to build without guessing
   - Implementation plan's task breakdown was granular enough

2. **Plan mode enforced discipline**
   - No "skipping ahead" to interesting parts
   - Dependencies respected (Epic 1 before Epic 2)

3. **Commit checkpoints reduced risk**
   - If something broke, easy to roll back to last Epic
   - Clear progress tracking in Git history

4. **Clarifying questions caught ambiguities early**
   - Better to answer 2-3 questions upfront than debug assumptions later

5. **Test execution at the end was effective**
   - Tests written during implementation (correct context)
   - Full suite run at end catches integration issues

### ⚠️ What Could Be Improved

1. **Test execution strategy**
   - Could run tests after each Epic (not just at end)
   - Catches issues earlier, reduces debugging at end

2. **MCP settings documentation**
   - Document which MCP tools were configured (for reproducibility)

3. **Claude.md review process**
   - Explicitly review `Claude.md` before saying "start building"
   - Confirm architecture understanding is correct

---

## Recommended Workflow for Future Projects

### Phase 1: Planning (Before Claude Code)
1. Complete requirements document
2. Define epics & stories
3. Write detailed implementation plan (tasks, subtasks, tests)
4. Document architecture decisions

### Phase 2: Claude Code Setup
1. Add standard agents and MCP settings file
2. Run `/init` in Claude Code
3. **Review `Claude.md`** (confirm understanding)
4. Ask: "Any clarifying questions before you start building?"

### Phase 3: Implementation (Epic by Epic)
1. Say: "Start building, beginning with Epic 0 (or Epic 1)"
2. Claude implements Epic (tasks → subtasks → tests)
3. **Run tests for completed Epic** (not deferred to end)
4. Commit and push (checkpoint)
5. Repeat for next Epic

### Phase 4: Integration Testing
1. After all Epics: "Run full test suite"
2. Claude runs all tests (unit, functional, integration, E2E)
3. Fix any failures
4. Commit final checkpoint

### Phase 5: Manual QA
1. Launch app manually (not just tests)
2. Smoke test critical paths (happy path + edge cases)
3. Document any issues for follow-up

---

## Tools Used

### Claude Code (Claude Desktop)
- **Version:** (specify version used)
- **Model:** Claude 3.5 Sonnet or Claude Opus
- **Mode:** Plan mode (structured implementation)

### MCP (Model Context Protocol)
- **Purpose:** Enable Claude Code to interact with local filesystem, Git, build tools
- **Configuration:** Standard agents + MCP settings file

### Repository
- **Platform:** GitHub
- **Branch strategy:** Feature branch per Epic (or all in one branch)
- **Commit strategy:** One commit per Epic

---

## Example: Epic Implementation Flow

**Epic 1: Foundation (Electron App Scaffold)**

1. **Claude reads Epic 1 goals** (from implementation plan)
2. **Implements tasks:**
   - Initialize Electron project
   - Configure TypeScript
   - Set up React + Vite
   - Create IPC bridge
   - Implement clean lifecycle
3. **Writes tests:**
   - Unit: Main process launches
   - Functional: Window opens, hot-reload works
   - Integration: IPC bidirectional
4. **Runs tests for Epic 1** (confirms passing)
5. **Commits:**
   ```bash
   git add .
   git commit -m "Epic 1: Foundation - Electron app scaffold, SQLite, IPC"
   git push
   ```
6. **Reports:** "Epic 1 complete. Ready for Epic 2?"

---

## Conclusion

**Key takeaway:** Plan mode + structured planning artifacts = rapid, reliable implementation.

**Jonathan's workflow demonstrates:**
- ✅ AI can implement complex projects autonomously (with good planning)
- ✅ Checkpoints (commits after every Epic) reduce risk
- ✅ Tests written during implementation (not deferred)
- ✅ Clarifying questions prevent misalignment
- ✅ `/init` creates shared understanding (human + AI on same page)

**For future projects:**
- Invest time in planning (requirements, epics, implementation plan)
- Use plan mode (structured > ad-hoc)
- Commit after every Epic (clear checkpoints)
- Run tests after each Epic (catch issues early)
- Review `Claude.md` before building (confirm understanding)

---

**Related Documents:**
- [Epics & Stories v2](epics-stories-v2.md)
- [Implementation Plan v2](pocket-consigliere-implementation-plan-v2.md)
- [Architecture Decision: Hybrid Approach](../processes/architecture-decisions.md)
- [Transcript: Hybrid Architecture Discussion](transcripts/2026-02-15-hybrid-architecture-discussion.md)

---

**Maintained by:** Jonathan Gough  
**Last updated:** 2026-02-15
