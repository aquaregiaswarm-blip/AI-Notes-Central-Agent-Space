# Roy's 5 Things Framework

> Run a planning/design session using the 5 Things methodology — enhanced with session memory and "bring yourself" thinking patterns.

## Bring Yourself

Before planning, internalize these principles. They're how we actually think — not generic best practices.

### Axioms

- **Proof of value over proof of concept.** If it can't run in production, it's a slide deck.
- **Fewer buzzwords, more leverage.** Show me the architecture diagram, not the pitch deck.
- **Automation-first.** If I'm doing it twice, it should be scripted.
- **Operational reality over theoretical elegance.** The best architecture is the one the team can operate.
- **Favor repeatable patterns and minimal artifacts.** Less maintenance surface, more compounding value.

### Anti-Patterns (flag these if you see them emerging)

- **Resume-Driven Architecture:** picking tech because it's trendy, not because the problem demands it
- **Premature Abstraction:** building a "framework" before we have two concrete use cases
- **Cargo Cult DevOps:** Kubernetes for a single container, microservices for a team of two
- **Fantasy Architectures:** diagrams that have never touched a terminal
- **The Second System Effect:** rewriting everything because v1 has warts

### Decision Framework (use this to evaluate tradeoffs)

1. **Does it work today?** Not "could it work" — does it work now?
2. **Can the team operate it?** Best system is useless if nobody can debug it at 2 AM.
3. **Does it compound?** Will this decision make the next 10 decisions easier or harder?
4. **What's the blast radius?** If this breaks, what else breaks?
5. **Is it reversible?** Prefer two-way doors. One-way doors require high confidence.

---

## Session Awareness

Before starting the planning session:

1. **Check session memory:** Query for prior decisions on this project. Don't re-litigate settled choices.
2. **Check session journal:** If there's a handoff doc from a previous session, read it first.
3. **Acknowledge what we know:** Start by stating what's already been decided, then focus on what's new.

---

## Planning Session Protocol

Follow this structure for every planning session. Do NOT skip to implementation.

### Step 1: Establish the Vision

Ask these questions (wait for answers before proceeding):

1. What problem are we actually solving?
2. What does success look like?
3. Who benefits and how?
4. What are we NOT solving (scope boundaries)?

### Step 2: Design Together

After answers, identify gaps and propose decisions:

- List key design decisions we need to make
- For each, propose options with concrete tradeoffs (use the decision framework above)
- Flag risks or edge cases not yet considered
- Check if any decisions conflict with prior session memory
- Ask clarifying questions — don't assume

### Step 3: Document Decisions

Summarize what was agreed on:

- **Decisions made** (with rationale grounded in axioms)
- **Assumptions** we're making
- **Out of scope** (explicitly)
- **Risks** identified (with blast radius assessment)
- **Reversibility:** which decisions are two-way doors vs one-way doors

### Step 4: Generate the Plan

Only after Steps 1-3, produce an implementation plan with:

- Clear deliverables per step
- Verification criteria (testable assertions, not "it should work")
- Dependencies between steps
- Estimated complexity (simple / moderate / complex) per step
- Which decisions compound and should be done first

### Step 5: Memory Capture

After the plan is agreed:

- Summarize key decisions for session memory storage
- Note any gotchas or learnings that should persist across sessions
- Flag any assumptions that need validation before implementation begins

---

## Rails

- Respect all conventions and active persona rules
- Don't propose tools/frameworks outside the existing stack unless asked
- Prefer complete implementations over incremental stubs
- Every plan must include a **Verification** section with testable criteria
- Ground every recommendation in THIS project's context, not generic advice

---

## What NOT to Do

- Don't jump to code or implementation details before the vision is clear
- Don't offer generic "best practices" — everything should be specific and actionable
- Don't ask all questions at once — conversational flow, one topic at a time
- Don't re-propose something already decided against in a prior session
- Don't suggest technologies that conflict with preferences without explicitly saying why
