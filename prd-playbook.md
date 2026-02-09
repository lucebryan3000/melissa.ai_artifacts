# PRD (Product Requirements Document) Playbook

> **Purpose**: Guide the creation of PRDs that clearly define the problem, users, and success criteria—aligning stakeholders on what to build and why
> **Version**: 1.0
> **Status**: Authoritative
> **Last Updated**: 2024-12-31

---

## Mental Model

A PRD is a contract between product thinking and execution. It answers "What problem are we solving and how will we know we've solved it?" before anyone writes code.

```
Problem → Users → Value → Success Criteria → Scope → Risks
   ↓        ↓       ↓            ↓             ↓        ↓
 "Why"   "Who"  "So what"   "Measure"     "Bounds"  "Watch out"
```

**The PRD Quality Ladder:**

| Level | State | Characteristics |
|-------|-------|-----------------|
| 0 | Wishful | "We need a dashboard" — solution, not problem |
| 1 | Directional | Problem mentioned but vague |
| 2 | Grounded | Problem clear, users fuzzy |
| 3 | Scoped | Problem + users + boundaries defined |
| 4 | Measurable | Success criteria are specific and testable |
| 5 | Actionable | Ready to decompose into FRD |

**Target: Level 4-5 before moving to functional requirements.**

**Critical Distinction:**
```
PRD: "Users struggle to find relevant reports, spending 20+ minutes searching"
     (Problem statement — describes pain)

NOT: "Build a search feature with filters and saved queries"
     (Solution statement — skips the why)
```

---

## Inputs / Outputs

### Inputs
- **Problem Signal**: User feedback, metrics, support tickets, market research
- **Business Context**: Strategy, goals, constraints
- **User Research**: Personas, journey maps, interviews
- **Competitive Landscape**: What alternatives exist
- **Technical Context**: Known constraints or opportunities

### Outputs
- **PRD Document**: Complete problem/solution framing
- **Success Metrics**: Measurable outcomes
- **Scope Boundaries**: What's in/out
- **Risk Register**: What could go wrong
- **Handoff**: Ready for FRD decomposition

---

## Evaluation Dimensions

### Dimension 1: Problem Clarity
The problem statement must be specific, falsifiable, and grounded in evidence. A vague problem leads to vague solutions.

**Probing Questions:**
- Can you state the problem in one sentence without mentioning a solution?
- What evidence proves this problem exists?
- How big is this problem? (users affected, frequency, severity)
- What happens if we don't solve this problem?
- Is this the root problem or a symptom of something deeper?

**Red Flags:**
- Problem statement includes solution ("We need X")
- No evidence cited
- Problem is too broad ("Users are frustrated")
- Problem is actually a feature request in disguise

**Good Example:**
> "Sales reps spend an average of 47 minutes per day manually entering CRM data from emails. This results in 15% data entry errors and delays pipeline visibility by 24 hours. Source: Time study (n=23), CRM audit Q3."

**Bad Example:**
> "We need to integrate email with the CRM"

---

### Dimension 2: Success Criteria
Outcomes must be measurable and unambiguous. If you can't measure success, you can't know if you've achieved it.

**Probing Questions:**
- What metric(s) will change if this succeeds?
- What's the baseline today?
- What's the target, and why that number?
- How will you measure it?
- What's the timeframe for achieving it?

**Red Flags:**
- Subjective criteria ("Users will be happier")
- No baseline measurement
- Unrealistic targets
- Unmeasurable outcomes

**Good Example:**
> **Primary Metric:** Reduce average daily CRM data entry time from 47 min to <15 min
> **Baseline:** 47 min/day (Q3 time study)
> **Target:** <15 min/day within 60 days of launch
> **Measurement:** Automated time tracking in CRM + monthly survey
>
> **Secondary Metric:** Reduce data entry error rate from 15% to <3%

**Bad Example:**
> "Users will find it easier to enter data"

---

### Dimension 3: Scope Boundaries
Clear boundaries prevent scope creep and set expectations. What's OUT is as important as what's IN.

**Probing Questions:**
- What's explicitly included in this effort?
- What's explicitly excluded?
- What adjacent problems might people assume are included?
- What's the MVP vs. nice-to-have?
- What would cause scope to expand, and how do we prevent it?

**Red Flags:**
- No explicit exclusions
- "And everything else users need"
- Scope defined by features, not outcomes
- No MVP distinction

**Good Example:**
> **In Scope:**
> - Automatic extraction of contact data from Gmail
> - Creation of CRM records from extracted data
> - Manual review/edit before commit
> - Gmail integration only (this phase)
>
> **Out of Scope:**
> - Outlook integration (Phase 2)
> - Calendar sync
> - Bi-directional sync (CRM → Email)
> - Mobile app support
>
> **MVP:** Gmail extraction + manual review. Auto-commit in v1.1.

**Bad Example:**
> "Email integration with CRM"

---

### Dimension 4: User/Actor Definition
Who experiences the problem? Who will use the solution? Ambiguous users lead to unfocused solutions.

**Probing Questions:**
- Who specifically experiences this problem?
- Are there multiple user types with different needs?
- What's the primary persona we're solving for?
- What are their current workarounds?
- Who else is affected (secondary users, admins, etc.)?

**Red Flags:**
- "Users" without specificity
- Multiple personas without prioritization
- No mention of current behavior
- Assumed user needs without research

**Good Example:**
> **Primary User:** Inside Sales Rep
> - 50 reps at company
> - Processes 80-120 emails/day
> - Current workaround: Copy-paste from email to CRM, takes ~30 sec/record
> - Pain: Tedious, error-prone, delays pipeline updates
>
> **Secondary User:** Sales Manager
> - Needs accurate pipeline data for forecasting
> - Currently sees 24-hour delay in data freshness
>
> **Not Addressed (this phase):** Field sales, customer success

**Bad Example:**
> "Sales team users"

---

### Dimension 5: Dependency Awareness
What must exist before this can be built? What will depend on this? Understanding dependencies prevents blocked work.

**Probing Questions:**
- What existing systems/data does this depend on?
- What decisions must be made before starting?
- What other initiatives depend on this?
- Are there external dependencies (vendors, APIs, approvals)?
- What technical prerequisites exist?

**Red Flags:**
- No dependencies listed (unlikely for real projects)
- Unvalidated assumptions about existing capabilities
- Circular dependencies
- External dependencies without contingency

**Good Example:**
> **Depends On:**
> - Gmail API access (approved, OAuth configured)
> - CRM API write access (available, rate limit: 100/min)
> - NLP service for entity extraction (evaluating options)
>
> **Blocks:**
> - Outlook integration (Phase 2) — same architecture
> - Sales automation initiative — needs reliable CRM data
>
> **Decisions Required:**
> - NLP vendor selection (deadline: Jan 15)
> - Data retention policy for extracted emails

**Bad Example:**
> (No dependencies section)

---

### Dimension 6: Risk Acknowledgment
What could go wrong? What's uncertain? Unacknowledged risks become surprise failures.

**Probing Questions:**
- What's most likely to go wrong?
- What's the worst-case outcome?
- What assumptions are we making that might be wrong?
- What external factors could derail this?
- What's the mitigation for top risks?

**Red Flags:**
- No risks listed
- Only technical risks (ignoring business, adoption)
- No mitigation strategies
- Risks without owners

**Good Example:**
> | Risk | Likelihood | Impact | Mitigation |
> |------|------------|--------|------------|
> | Gmail API rate limits hit during peak | Medium | High | Implement queuing, request limit increase |
> | Users don't trust auto-extraction accuracy | High | High | Start with human-in-loop, build trust |
> | NLP extraction accuracy <90% | Medium | High | Fallback to manual entry, improve model |
> | Low adoption ("I'll just do it manually") | Medium | Medium | Gamification, manager visibility |

**Bad Example:**
> "There might be some technical challenges"

---

## Extended Question Bank

### Problem Discovery
1. If you had to explain this problem to a new employee in one sentence, what would you say?
2. What's the cost of this problem today? (time, money, errors, churn)
3. How do you know this is a real problem vs. assumed problem?
4. What evidence would convince a skeptic this problem matters?
5. Is this the root problem or a symptom? (Ask "why" 5 times)

### Success Definition
6. What does "success" look like in 30 days? 90 days? 1 year?
7. What metric moving would make you confident this worked?
8. What's the minimum improvement that would justify the investment?
9. How will you avoid vanity metrics?
10. What leading indicators will show early progress?

### User Understanding
11. Who wakes up in the morning with this problem?
12. What do they do today to work around it?
13. Have you talked to actual users about this? What did they say?
14. What would make users NOT adopt a solution?
15. Are there users who benefit from the current broken state?

### Scope Control
16. If you could only ship one thing, what would it be?
17. What would you cut if timeline was halved?
18. What's the most common scope creep request you'll hear?
19. How will you say no to out-of-scope requests?
20. What's the difference between MVP and v1.0?

### Dependency Mapping
21. What APIs/services does this touch?
22. What team dependencies exist?
23. What needs to be true in 3 months that isn't true today?
24. What vendor decisions are outstanding?
25. What approvals are needed (legal, security, compliance)?

### Risk Assessment
26. What would cause this initiative to fail completely?
27. What are you assuming that might not be true?
28. What's the adoption risk? (build it, they don't come)
29. What competitive or market risks exist?
30. What's your contingency if the primary approach fails?

---

## Common Pitfalls

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| **Solution as problem** | PRD describes features, not pain | Rewrite: "Users struggle to..." not "We need to build..." |
| **Unmeasurable success** | "Improve experience" | Add specific metric + target + timeframe |
| **Missing users** | "Users will..." without specificity | Name the persona, describe their day |
| **Infinite scope** | Everything is in scope | Add explicit "Out of Scope" section |
| **No evidence** | Problem assumed, not proven | Add data, quotes, research citations |
| **Wishful metrics** | Targets without baselines | Measure current state before setting targets |
| **Risk blindness** | No risks listed | Assume things will go wrong, list them |
| **Dependency amnesia** | "We just need to build it" | Map every external dependency |
| **Premature solutioning** | PRD prescribes implementation | Stay in problem space, FRD handles solution |
| **Stakeholder misalignment** | Different people read different PRDs | Single source of truth, version controlled |

---

## PRD Template

```markdown
# PRD: [Feature/Initiative Name]

> **Owner:** [Name]
> **Status:** [Draft | Review | Approved]
> **Version:** [X.X]
> **Last Updated:** [YYYY-MM-DD]

---

## Executive Summary

[2-3 sentences: What problem, for whom, expected outcome]

---

## Problem Statement

### The Problem
[Clear, evidence-based description of the pain point]

### Evidence
- [Data point 1]
- [User quote / research finding]
- [Business impact metric]

### Root Cause
[Why does this problem exist?]

### Cost of Inaction
[What happens if we don't solve this?]

---

## Users & Stakeholders

### Primary User
**[Persona Name]**
- Role: [Job title/function]
- Population: [How many]
- Current Behavior: [What they do today]
- Pain: [How problem affects them]

### Secondary Users
- [User type]: [Brief description]

### Stakeholders
- [Stakeholder]: [Their interest]

---

## Success Criteria

### Primary Metric
- **Metric:** [What you're measuring]
- **Baseline:** [Current state]
- **Target:** [Goal]
- **Timeframe:** [When]
- **Measurement:** [How you'll measure]

### Secondary Metrics
- [Metric 2]
- [Metric 3]

### Non-Goals
- [What success is NOT]

---

## Scope

### In Scope
- [Capability 1]
- [Capability 2]

### Out of Scope
- [Explicit exclusion 1]
- [Explicit exclusion 2]

### MVP Definition
[Minimum viable version]

### Future Phases
- Phase 2: [Description]
- Phase 3: [Description]

---

## Dependencies

### Technical Dependencies
- [System/API]: [Status]

### Team Dependencies
- [Team]: [What you need from them]

### External Dependencies
- [Vendor/Partner]: [Status]

### Decisions Required
- [ ] [Decision 1] — Owner: [Name], Deadline: [Date]

---

## Risks

| Risk | Likelihood | Impact | Mitigation | Owner |
|------|------------|--------|------------|-------|
| [Risk] | H/M/L | H/M/L | [Action] | [Name] |

---

## Open Questions

- [ ] [Question 1]
- [ ] [Question 2]

---

## Appendix

### Research & Evidence
[Links to research, data, user interviews]

### Related Documents
- [FRD]: [Link]
- [Technical Spec]: [Link]

---

## Changelog

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| X.X | YYYY-MM-DD | [Name] | [Changes] |
```

---

## Invariants

1. **Problem statements MUST NOT contain solutions** — describe pain, not features
2. **Success criteria MUST be measurable** — metric + baseline + target + timeframe
3. **Users MUST be specifically identified** — not "users" but named personas with context
4. **Scope MUST have explicit exclusions** — what's OUT prevents creep
5. **Evidence MUST support the problem** — data, quotes, research, not assumptions
6. **Risks MUST be acknowledged** — no PRD is risk-free
7. **Dependencies MUST be mapped** — nothing exists in isolation
8. **PRD MUST stay in problem space** — functional details belong in FRD

---

## Artifact Lineage

```
User Research / Business Need
        ↓
      PRD  ← (You are here)
        ↓
      FRD
        ↓
   Technical Spec
        ↓
 Implementation Plan
```

**Handoff to FRD:**
- Problem is clear and evidence-based
- Success criteria are measurable
- Scope is bounded
- Users are defined
- Ready to decompose into functional requirements

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2024-12-31 | Initial playbook: 6 dimensions, 30-question bank, template, invariants |

---

*Product Thinker — Defining the problem worth solving before jumping to solutions.*
