# ADR (Architecture Decision Record) Playbook

> **Purpose**: Guide the creation of Architecture Decision Records that capture significant decisions with context, rationale, and consequences
> **Version**: 1.0
> **Last Updated**: 2024-12-31

---

## Mental Model

An ADR is a point-in-time snapshot of a decision. It answers "Why did we decide this?" for anyone who comes later—including future you. Good ADRs make the implicit explicit.

```
Context → Decision → Rationale → Consequences
    ↓         ↓          ↓            ↓
 "Situation"  "Choice"  "Why this"  "What follows"
```

**The Decision Lifecycle:**
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  PROPOSED   │───▶│  ACCEPTED   │───▶│ SUPERSEDED  │
└─────────────┘    └─────────────┘    └─────────────┘
                          │                  ▲
                          │                  │
                          ▼                  │
                   ┌─────────────┐           │
                   │ DEPRECATED  │───────────┘
                   └─────────────┘
```

**When to Write an ADR:**
- Choosing technologies (languages, frameworks, databases)
- Defining architectural patterns (microservices, event-driven)
- Making integration decisions (API design, protocols)
- Setting standards (code style, testing strategy)
- Any decision that's hard to reverse
- Any decision people keep asking about

**The ADR Quality Ladder:**

| Level | State | Characteristics |
|-------|-------|-----------------|
| 0 | Missing | Decision in someone's head or Slack |
| 1 | Noted | "We decided X" — no context |
| 2 | Explained | Context exists, rationale weak |
| 3 | Justified | Alternatives considered, rationale solid |
| 4 | Complete | Consequences documented |
| 5 | Linked | Connected to related ADRs and artifacts |

**Target: Level 4-5 for all significant decisions.**

---

## Inputs / Outputs

### Inputs
- **Decision Need**: Problem or opportunity requiring a choice
- **Context**: Technical and business situation
- **Options**: Alternatives being considered
- **Constraints**: What limits the decision space
- **Stakeholders**: Who is affected and who decides

### Outputs
- **ADR Document**: Captured decision with rationale
- **Decision Log Entry**: Summary for decision register
- **Related Updates**: Links from affected artifacts

---

## Evaluation Dimensions

### Dimension 1: Decision Clarity
The decision itself must be unambiguous. A reader should know exactly what was decided.

**Probing Questions:**
- Can you state the decision in one sentence?
- Is the decision actionable?
- Is there any ambiguity in what was decided?
- Does the decision have a clear scope?
- Would two people interpret this decision the same way?

**Red Flags:**
- "We will consider using X"
- Decision requires interpretation
- Multiple decisions in one ADR
- Decision scope unclear

**Good Example:**
> **Decision:** We will use PostgreSQL as the primary database for the Orders service, deployed as AWS RDS in Multi-AZ configuration.

**Bad Example:**
> **Decision:** We will use a robust database solution.

---

### Dimension 2: Context Completeness
Why did this decision need to be made? What situation triggered it? Without context, rationale makes no sense.

**Probing Questions:**
- What problem or opportunity triggered this decision?
- What's the current state?
- What constraints exist (technical, business, time)?
- Who are the stakeholders affected?
- What's the timeline pressure?

**Red Flags:**
- "We needed a database"
- Missing business context
- No mention of constraints
- Context written after decision

**Good Example:**
```
## Context

The Orders service is being extracted from the monolith as part of our 
microservices migration (see ADR-012). We need a dedicated database that:

- Handles 10,000 writes/minute at peak
- Supports ACID transactions for order integrity
- Integrates with our existing AWS infrastructure
- Can be operated by a team with SQL expertise (no dedicated DBA)
- Must be production-ready within 8 weeks

Currently, order data lives in a shared Oracle database that is a 
scaling bottleneck and single point of failure. The Oracle license 
costs $200K/year.
```

**Bad Example:**
```
## Context
We need a database for orders.
```

---

### Dimension 3: Options Considered
What alternatives were genuinely evaluated? A decision without alternatives isn't a decision—it's a foregone conclusion.

**Probing Questions:**
- What options were seriously considered?
- Was the status quo considered?
- What made each option viable or not?
- Were options evaluated fairly?
- Who proposed each option?

**Red Flags:**
- Only one option listed
- "We didn't consider alternatives"
- Strawman options (clearly inferior)
- No evaluation criteria

**Good Example:**
```
## Options Considered

### Option 1: PostgreSQL (RDS)
- Mature, SQL-compatible, team has expertise
- Managed service reduces ops burden
- Cost: ~$15K/year for Multi-AZ
- Tradeoff: Less scalable than NoSQL options

### Option 2: Amazon Aurora
- MySQL/PostgreSQL compatible, better scaling
- Higher cost: ~$25K/year
- Tradeoff: Less transparent pricing, vendor lock-in

### Option 3: MongoDB Atlas
- Flexible schema, good for evolving data
- Team would need training
- Cost: ~$20K/year
- Tradeoff: Weaker transaction support, new tech

### Option 4: Stay on Oracle (Status Quo)
- No migration effort
- Maintains shared database problems
- Cost: $200K/year (allocated share)
- Tradeoff: Doesn't solve bottleneck
```

**Bad Example:**
```
## Options
We considered PostgreSQL and decided to use it.
```

---

### Dimension 4: Rationale Strength
Why THIS option over others? The rationale must be compelling and traceable. "Because" isn't a reason.

**Probing Questions:**
- What criteria drove the decision?
- How did the chosen option score against criteria?
- What was the deciding factor?
- Is the rationale logical and traceable?
- Would a skeptic be convinced?

**Red Flags:**
- "It seemed like the best choice"
- "The team preferred it"
- No evaluation criteria
- Rationale doesn't connect to context

**Good Example:**
```
## Decision

We will use PostgreSQL (AWS RDS) for the Orders service database.

## Rationale

**Evaluation Criteria (weighted):**
| Criterion | Weight | Postgres | Aurora | MongoDB | Oracle |
|-----------|--------|----------|--------|---------|--------|
| Team expertise | 30% | 5 | 4 | 2 | 4 |
| Operational cost | 25% | 5 | 3 | 4 | 1 |
| Transaction support | 20% | 5 | 5 | 3 | 5 |
| Scaling ceiling | 15% | 3 | 5 | 5 | 4 |
| Migration effort | 10% | 4 | 4 | 2 | 5 |
| **Weighted Score** | | **4.4** | **4.0** | **3.1** | **3.3** |

**Key Deciding Factors:**
1. Team expertise: Senior engineers have 5+ years PostgreSQL experience. 
   MongoDB would require 3-month training investment.
2. Cost: PostgreSQL RDS is 85% cheaper than Oracle allocation
3. Transaction support: Order integrity requires ACID—MongoDB's 
   transaction model adds complexity

**Why not Aurora:** While Aurora offers better scaling, our projected 
load (10K writes/min) is well within PostgreSQL limits. Aurora's 
premium isn't justified until we hit scaling needs we don't anticipate 
for 2+ years.
```

**Bad Example:**
```
## Rationale
PostgreSQL is a good database and the team likes it.
```

---

### Dimension 5: Consequences Acknowledged
Every decision has tradeoffs. What do we gain? What do we give up? What new challenges does this create?

**Probing Questions:**
- What are the positive consequences?
- What are the negative consequences?
- What risks does this decision introduce?
- What decisions does this enable or constrain?
- What needs to happen as a result?

**Red Flags:**
- Only positive consequences listed
- "No significant downsides"
- No mention of risks
- No action items

**Good Example:**
```
## Consequences

### Positive
- Team can be productive immediately (no learning curve)
- 85% cost reduction vs Oracle ($170K/year savings)
- AWS RDS handles backups, patching, failover
- Strong ecosystem of tools and extensions
- Easier hiring (PostgreSQL skills common)

### Negative
- Scaling ceiling lower than Aurora/MongoDB (acceptable for 2+ years)
- Must manage read replicas manually for read scaling
- No automatic sharding (would need Citus extension)
- Vendor lock-in to AWS RDS (acceptable tradeoff)

### Risks
| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Write volume exceeds capacity | Low | High | Monitor, plan Aurora migration path |
| RDS outage | Low | High | Multi-AZ, automated failover |
| Schema evolution pain | Medium | Medium | Careful migration planning |

### Follow-up Actions
- [ ] Provision RDS instances (DevOps, Week 1)
- [ ] Set up monitoring dashboards (DevOps, Week 1)
- [ ] Create migration scripts from Oracle (Backend, Weeks 2-4)
- [ ] Document connection patterns (Backend, Week 2)
- [ ] Load testing to validate capacity (QA, Week 6)
```

**Bad Example:**
```
## Consequences
This will give us a better database.
```

---

### Dimension 6: Reversibility
How hard is it to undo this decision? Highly reversible decisions need less scrutiny; irreversible ones need more.

**Probing Questions:**
- How hard is it to reverse this decision?
- What's the cost of reversal?
- At what point does reversal become impractical?
- Under what conditions would we reconsider?
- How do we know if this decision was wrong?

**Red Flags:**
- No mention of reversibility
- "This is permanent"
- No triggers for reconsideration
- No success metrics

**Good Example:**
```
## Reversibility

**Reversibility Level:** Medium

**Reversal Cost:**
- Time: 2-3 months for full migration
- Effort: ~500 engineering hours
- Risk: Data migration complexity

**Point of No Return:**
Once Orders service is in production with customer data, switching 
databases requires careful migration. The decision becomes harder to 
reverse after:
- Production launch (Week 8)
- 6 months of production data accumulation

**Reconsideration Triggers:**
- Write volume consistently >80% of capacity for 2+ weeks
- RDS availability <99.9% over any quarter
- Team productivity significantly impacted by PostgreSQL limitations

**Success Metrics:**
| Metric | Target | Review |
|--------|--------|--------|
| Write latency p99 | <50ms | Weekly |
| Availability | >99.9% | Monthly |
| Cost vs Oracle | >80% savings | Quarterly |
| Team satisfaction | >4/5 | Quarterly survey |
```

**Bad Example:**
```
## Reversibility
We can change it later if needed.
```

---

## Extended Question Bank

### Decision Clarity
1. State the decision in one sentence—is it clear?
2. What exactly is being decided?
3. What is NOT being decided (deferred)?
4. Is this one decision or multiple?
5. What's the scope of this decision?

### Context
6. What triggered this decision?
7. What's the current state?
8. What constraints limit options?
9. Who are the stakeholders?
10. What's the timeline?

### Options
11. What options were seriously considered?
12. Was "do nothing" an option?
13. What criteria were used to evaluate options?
14. Were any options unfairly dismissed?
15. Who proposed each option?

### Rationale
16. What was the deciding factor?
17. Can you score options against criteria?
18. Would a skeptic accept this rationale?
19. Is the logic traceable?
20. What assumptions are we making?

### Consequences
21. What do we gain from this decision?
22. What do we give up or accept as risk?
23. What does this enable or prevent?
24. What follow-up actions are needed?
25. Who needs to be informed?

### Reversibility
26. How hard is it to change course?
27. What's the cost of being wrong?
28. When does this become irreversible?
29. Under what conditions would we reconsider?
30. How will we know if this was the right decision?

---

## Common Pitfalls

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| **Written after the fact** | Context feels reconstructed | Write ADR when making decision |
| **No alternatives** | "We decided X" | Document at least 3 options |
| **Weak rationale** | "It seemed best" | Score options against criteria |
| **Missing context** | Reader asks "but why?" | Include business + technical context |
| **Only positives** | No consequences or risks | Document tradeoffs honestly |
| **Too long** | Nobody reads it | Keep focused, link to details |
| **Too vague** | Ambiguous decision | Make decision specific and actionable |
| **Not findable** | Decisions lost in docs | Index in decision log |
| **Never updated** | Stale decisions | Mark superseded, link to new ADR |
| **Single author** | Missing perspectives | Review with stakeholders |

---

## ADR Template

```markdown
# ADR-[NNN]: [Short Title]

> **Status:** [Proposed | Accepted | Deprecated | Superseded]
> **Date:** [YYYY-MM-DD]
> **Deciders:** [Names/roles]
> **Supersedes:** [ADR-XXX if applicable]
> **Superseded by:** [ADR-XXX if applicable]

---

## Summary

[One paragraph summary of context and decision]

---

## Context

[What is the situation? What problem or opportunity triggered this decision?]

### Current State
[What exists today]

### Problem Statement
[What specific problem are we solving]

### Constraints
- [Constraint 1]
- [Constraint 2]

### Stakeholders
- [Who is affected]
- [Who has input]
- [Who decides]

---

## Decision

[Clear, unambiguous statement of what we're doing]

**We will:** [Specific action]

**We will not:** [Explicit exclusions if helpful]

---

## Options Considered

### Option 1: [Name]
- **Description:** [What this option is]
- **Pros:** [Benefits]
- **Cons:** [Drawbacks]
- **Estimated Effort:** [Time/cost]

### Option 2: [Name]
[Same structure]

### Option 3: [Name] (Status Quo)
[Same structure]

---

## Rationale

### Evaluation Criteria
| Criterion | Weight | Option 1 | Option 2 | Option 3 |
|-----------|--------|----------|----------|----------|
| [Criterion] | [%] | [Score] | [Score] | [Score] |

### Key Deciding Factors
1. [Most important factor and how it drove decision]
2. [Second factor]
3. [Third factor]

### Why Not [Rejected Option]?
[Specific reason the runner-up was rejected]

---

## Consequences

### Positive
- [Benefit 1]
- [Benefit 2]

### Negative
- [Tradeoff 1]
- [Tradeoff 2]

### Risks
| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| [Risk] | L/M/H | L/M/H | [Action] |

### Follow-up Actions
- [ ] [Action 1] — Owner, Deadline
- [ ] [Action 2] — Owner, Deadline

---

## Reversibility

**Level:** [Easy | Medium | Hard | Irreversible]

**Cost of Reversal:**
[What it would take to undo this]

**Point of No Return:**
[When reversal becomes impractical]

**Reconsideration Triggers:**
- [Condition that would trigger re-evaluation]

---

## Success Metrics

| Metric | Target | Review Frequency |
|--------|--------|------------------|
| [Metric] | [Target] | [Weekly/Monthly] |

---

## References

- [Link to related PRD, spec, or ADR]
- [Link to research or benchmarks]
- [Link to meeting notes or discussions]

---

## Changelog

| Date | Author | Changes |
|------|--------|---------|
| [Date] | [Name] | Initial draft |
| [Date] | [Name] | [Update description] |
```

---

## ADR Numbering & Organization

### Numbering Scheme
```
ADR-001  First decision
ADR-002  Second decision
...
ADR-NNN  Sequential, never reused
```

### File Naming
```
docs/adr/
├── ADR-001-use-postgresql-for-orders.md
├── ADR-002-adopt-event-driven-architecture.md
├── ADR-003-select-kubernetes-for-orchestration.md
├── README.md  (index of all ADRs)
└── _template.md
```

### Decision Log (README.md)
```markdown
# Architecture Decision Records

| ADR | Title | Status | Date |
|-----|-------|--------|------|
| [ADR-001](ADR-001-use-postgresql.md) | Use PostgreSQL for Orders | Accepted | 2024-01-15 |
| [ADR-002](ADR-002-event-driven.md) | Adopt Event-Driven Architecture | Accepted | 2024-02-01 |
| [ADR-003](ADR-003-kubernetes.md) | Select Kubernetes | Superseded by ADR-007 | 2024-02-15 |
```

---

## Invariants

1. **Every significant decision MUST have an ADR** — if people keep asking, write it down
2. **ADRs MUST be written when deciding, not after** — reconstructed context is incomplete
3. **Every ADR MUST have alternatives** — single-option ADRs aren't decisions
4. **Rationale MUST be explicit** — "because" isn't a reason
5. **Consequences MUST include negatives** — every decision has tradeoffs
6. **ADRs are IMMUTABLE** — never edit accepted ADRs, supersede with new ones
7. **Superseded ADRs MUST link to replacement** — maintain traceability
8. **ADRs MUST be findable** — index in decision log

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2024-12-31 | Initial playbook: 6 dimensions, 30-question bank, template, invariants |

---

*Decision Analyst — Capturing the "why" so future teams don't repeat history.*
