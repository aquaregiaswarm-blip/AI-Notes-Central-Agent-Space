# User Requirements Gathering Notes

*Techniques for extracting user requirements, focusing on personas and user journeys.*

---

## 1. Persona Development

### Data Gathering Methods

| Technique | Best For | Output |
|-----------|----------|--------|
| **User Interviews** | Deep qualitative insights | Goals, frustrations, quotes |
| **Surveys** | Quantitative patterns at scale | Demographics, preferences |
| **Contextual Inquiry** | Observing real behavior | Workflows, workarounds |
| **Analytics/CRM Data** | Behavioral patterns | Segments, usage patterns |
| **Support Tickets** | Pain points | Common issues, language |

### Persona Components

- **Demographics** — Role, seniority, industry
- **Goals** — What they're trying to achieve
- **Pain Points** — Frustrations with current state
- **Behaviors** — How they work today
- **Motivations** — Why they do what they do
- **Quote** — Captures their voice
- **Day in the Life** — Context for their work

### Frameworks

- **Jobs-to-be-Done (JTBD)** — Focus on the "job" the user is hiring the product to do
- **Empathy Maps** — Think/Feel/Say/Do quadrants
- **Proto-Personas** — Quick hypothesis-based personas (validate later)

---

## 2. User Journey Mapping

### Journey Map Components

1. **Stages** — Awareness → Consideration → Decision → Onboarding → Usage → Advocacy
2. **Actions** — What the user does at each stage
3. **Touchpoints** — Where they interact with your product/service
4. **Thoughts** — What they're thinking
5. **Emotions** — How they feel (the emotional curve)
6. **Pain Points** — Friction at each stage
7. **Opportunities** — Where you can improve

### Techniques

| Method | Description |
|--------|-------------|
| **Current State Map** | Document how things work today |
| **Future State Map** | Envision the ideal experience |
| **Service Blueprint** | Add backstage processes (what supports the journey) |
| **Experience Map** | Broader context beyond your product |

### Workshop Format

1. Define the persona
2. List all touchpoints
3. Walk through chronologically
4. Capture emotions at each step
5. Identify moments of truth (high impact)
6. Mark pain points and opportunities

---

## 3. Requirements Extraction from Journeys

### Convert Journey → Requirements

```
Pain Point: "User has to manually re-enter data from CRM"
     ↓
User Story: "As a Sales Rep, I want CRM data auto-populated 
             so I don't waste time on data entry"
     ↓
Requirement: CRM integration with bi-directional sync
```

### Prioritization Criteria

- **Impact** — How many users? How painful?
- **Frequency** — How often does this occur?
- **Effort** — How hard to solve?

---

## 4. Quick Reference: Technique Selection

| If you have... | Try this |
|----------------|----------|
| **No users yet** | Proto-personas + assumption mapping |
| **Limited time** | 5-8 interviews + affinity mapping |
| **Existing product** | Analytics + support ticket analysis |
| **B2B product** | Day-in-the-life shadowing |
| **Multiple user types** | Separate journey per persona |

---

## 5. Persona Template

```markdown
## [Persona Name]

**Role:** [Job title]
**Industry:** [Sector]
**Experience:** [Years in role]

### Goals
- [Primary goal]
- [Secondary goal]

### Pain Points
- [Frustration 1]
- [Frustration 2]

### Behaviors
- [How they work today]
- [Tools they use]

### Quote
> "[Something they might say that captures their mindset]"

### Day in the Life
[Brief narrative of typical workday]
```

---

## 6. Journey Map Template

```markdown
## [Persona Name] Journey: [Journey Name]

### Stage 1: [Stage Name]
- **Actions:** [What user does]
- **Touchpoints:** [Where interaction happens]
- **Thoughts:** [What they're thinking]
- **Emotions:** [😊 😐 😞]
- **Pain Points:** [Friction]
- **Opportunities:** [How to improve]

### Stage 2: [Stage Name]
[Repeat structure]
```

---

## Resources

- Nielsen Norman Group — Journey mapping guides
- IDEO — Human-centered design toolkit
- Strategyzer — Value Proposition Canvas
- Intercom — Jobs-to-be-Done framework
