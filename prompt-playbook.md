# Prompt / Agent Spec Playbook

> **Purpose**: Guide the creation, refinement, and evaluation of prompts and agent specifications for AI-assisted coding workflows
> **Version**: 1.0
> **Last Updated**: 2024-12-31

---

## Mental Model

A prompt is a contract between you and an AI agent. The clearer the contract, the better the output.

```
Intent → Constraints → Context → Instructions → Output Contract → Evaluation
   ↓          ↓           ↓            ↓              ↓              ↓
 "What"    "What NOT"  "With what"  "How"        "Shape"      "Did it work?"
```

**The Prompt Quality Ladder:**

| Level | State | Characteristics |
|-------|-------|-----------------|
| 0 | Vague | "Help me with code" — no goal, no constraints |
| 1 | Directional | Goal exists but fuzzy; no boundaries |
| 2 | Scoped | Clear goal + explicit out-of-scope |
| 3 | Contracted | Input/output shapes defined; constraints explicit |
| 4 | Robust | Failure modes handled; edge cases addressed |
| 5 | Evaluable | Success criteria defined; testable outcomes |

**Target: Level 4-5 for any production prompt.**

---

## Inputs / Outputs

### Inputs
- **Intent**: What should the agent accomplish?
- **Context**: What information/files/state does the agent receive?
- **Constraints**: What must the agent NOT do?
- **Environment**: Where will this run? (Claude Code, Codex CLI, API, etc.)
- **Examples**: Sample inputs/outputs if available

### Outputs
- **Prompt/Agent Spec**: Complete, production-ready prompt document
- **Evaluation Criteria**: How to test if the prompt works
- **Failure Modes**: Known edge cases and how to handle them

---

## Evaluation Dimensions

### Dimension 1: Goal Clarity
The prompt's objective must be unambiguous. A reader (human or AI) should understand exactly what success looks like.

**Probing Questions:**
- Can you state the goal in one sentence?
- Is the goal falsifiable? (Can you tell if it succeeded or failed?)
- Are there hidden sub-goals that should be explicit?
- Does the goal have measurable outcomes?

**Red Flags:**
- "Help with..." (too vague)
- Multiple competing goals
- Goals that can't be verified

**Good Example:**
> "Generate a TypeScript function that validates email addresses against RFC 5322, returning `{ valid: boolean, reason?: string }`"

**Bad Example:**
> "Help me with email validation"

---

### Dimension 2: Constraint Boundaries
What the agent must NOT do is as important as what it should do. Constraints prevent scope creep, unsafe behavior, and wasted effort.

**Probing Questions:**
- What actions are explicitly forbidden?
- What's out of scope for this prompt?
- Are there safety/security constraints?
- What resources should the agent NOT access?
- Are there token/length/time constraints?

**Red Flags:**
- No explicit constraints (anything goes)
- Constraints conflict with the goal
- Over-constrained (impossible to complete)

**Good Example:**
> "Do NOT modify files outside `/src`. Do NOT install new dependencies. Do NOT make network requests."

**Bad Example:**
> (No constraints specified — agent might do anything)

---

### Dimension 3: Input Expectations
The agent needs to know what context it receives. Ambiguous inputs produce ambiguous outputs.

**Probing Questions:**
- What data/files/state does the agent receive?
- What format is the input? (JSON, markdown, free text, files?)
- Are there required vs. optional inputs?
- What if input is malformed or missing?
- Is there session/conversation context?

**Red Flags:**
- Input format undefined
- Required fields not specified
- No handling for missing/bad input

**Good Example:**
> "Input: A markdown file containing a PRD. Required sections: Problem Statement, Success Criteria. Optional: User Stories, Risks."

**Bad Example:**
> "Input: the document"

---

### Dimension 4: Output Contract
The agent's output must have a defined shape. Without this, downstream consumers can't rely on the output.

**Probing Questions:**
- What format should the output be? (JSON, markdown, code, file?)
- Is there a schema or structure to follow?
- What fields are required vs. optional?
- Are there length/size constraints?
- Should output be streamed or complete?

**Red Flags:**
- Output format undefined
- Multiple possible formats with no selection criteria
- No schema for structured output

**Good Example:**
> "Output: JSON object with structure: `{ summary: string, decisions: Array<{ id: string, description: string, rationale: string }>, next_steps: string[] }`"

**Bad Example:**
> "Output: a response"

---

### Dimension 5: Failure Behavior
Agents fail. The prompt should define what happens when things go wrong.

**Probing Questions:**
- What if the agent can't complete the task?
- What if input is invalid?
- What if the agent hits a constraint boundary?
- Should failures be silent, logged, or escalated?
- Is there a fallback behavior?

**Red Flags:**
- No failure handling defined
- Agent might silently fail or hallucinate
- No escalation path

**Good Example:**
> "If the PRD is missing a Problem Statement section, stop and report: `{ error: 'MISSING_REQUIRED_SECTION', section: 'Problem Statement' }`. Do not proceed without required sections."

**Bad Example:**
> (No failure handling — agent might guess or hallucinate)

---

### Dimension 6: Evaluation Criteria
You need to know if the prompt worked. Testable success criteria make prompts maintainable.

**Probing Questions:**
- How do you verify the output is correct?
- Are there automated tests or checks?
- What does "good enough" look like?
- How do you measure quality over multiple runs?
- Is there a human review step?

**Red Flags:**
- No way to test success
- Subjective criteria only ("looks good")
- No quality thresholds

**Good Example:**
> "Success criteria: (1) Output parses as valid JSON, (2) All required fields present, (3) Summary is <200 words, (4) At least 3 decisions captured. Automated validation: run `validate-output.ts`."

**Bad Example:**
> "Check if it looks right"

---

## Extended Question Bank

### Goal Refinement
1. If you had to explain this prompt's purpose to a new team member in one sentence, what would you say?
2. What's the single most important thing this agent must get right?
3. What would make you reject the output as a failure?
4. Is this prompt trying to do too many things at once?
5. What decision does the output of this prompt enable?

### Constraint Discovery
6. What's the worst thing this agent could do if unconstrained?
7. Are there any "never do this" rules from your codebase or workflow?
8. What external systems should this agent never touch?
9. What happens if the agent takes too long?
10. Are there security or compliance boundaries?

### Input Clarity
11. Walk me through a typical input — what does it look like?
12. What variations of input might this agent receive?
13. What's the minimum viable input to produce useful output?
14. How would you handle input in a different language?
15. What context from prior conversation is relevant?

### Output Precision
16. Who or what consumes this output?
17. If another system parses this output, what schema does it expect?
18. What's the maximum acceptable output length?
19. Should the output include reasoning or just conclusions?
20. How does the output connect to the next step in your workflow?

### Failure Resilience
21. What's the most likely way this prompt will fail?
22. If the agent is uncertain, should it ask for help or make a best guess?
23. What's the recovery path if output is wrong?
24. Should partial output be returned, or all-or-nothing?
25. How do you detect that the agent is stuck in a loop?

### Evaluation Design
26. Can you write a unit test for this prompt's output?
27. What's your quality bar — 80% good enough, or 99% required?
28. How often will you review outputs manually?
29. What metrics would indicate this prompt is degrading over time?
30. Is there a golden dataset of input/output pairs to validate against?

---

## Common Pitfalls

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| **Vague goal** | Output is unfocused or off-target | Rewrite goal as falsifiable statement |
| **Missing constraints** | Agent does unexpected things | Add explicit "DO NOT" section |
| **Scope creep** | Prompt grows to handle everything | Split into multiple focused prompts |
| **Over-specification** | Prompt is brittle, breaks on edge cases | Relax unnecessary constraints |
| **Under-specification** | Output varies wildly between runs | Add structure to output contract |
| **No failure path** | Agent hallucinates or hangs | Define explicit error handling |
| **Untestable** | Can't verify if prompt works | Add measurable success criteria |
| **Context overload** | Too much context, agent gets confused | Trim to essential context only |
| **Assumed knowledge** | Agent doesn't know what you know | Make implicit knowledge explicit |
| **Missing examples** | Agent guesses output format | Add 1-2 concrete examples |

---

## Prompt Template

```markdown
# [Agent Name]

> **Purpose**: [One sentence goal]
> **Environment**: [Claude Code / Codex CLI / API / etc.]
> **Version**: [X.X]

---

## Goal

[Clear, falsifiable statement of what this agent accomplishes]

---

## Constraints

**MUST:**
- [Required behavior 1]
- [Required behavior 2]

**MUST NOT:**
- [Forbidden action 1]
- [Forbidden action 2]

**SHOULD:**
- [Preferred behavior 1]

---

## Inputs

| Input | Type | Required | Description |
|-------|------|----------|-------------|
| [name] | [type] | Yes/No | [description] |

**Input Format:**
[Describe expected format, schema, or structure]

**Handling Missing/Invalid Input:**
[What happens if input is bad]

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| [name] | [type] | [description] |

**Output Format:**
[Schema, structure, or example]

---

## Failure Handling

| Failure Mode | Detection | Response |
|--------------|-----------|----------|
| [failure type] | [how to detect] | [what to do] |

---

## Evaluation Criteria

- [ ] [Testable criterion 1]
- [ ] [Testable criterion 2]
- [ ] [Testable criterion 3]

**Validation Method:**
[How to verify success — automated test, manual review, etc.]

---

## Examples

### Example 1: [Scenario Name]

**Input:**
```
[sample input]
```

**Expected Output:**
```
[sample output]
```

---

## Notes

[Any additional context, gotchas, or evolution notes]
```

---

## Invariants

1. **Every prompt MUST have a falsifiable goal** — if you can't tell success from failure, rewrite it
2. **Every prompt MUST have explicit constraints** — unbounded agents are dangerous agents
3. **Every prompt MUST define output format** — consumers need to know what to expect
4. **Failure handling MUST be defined before production use** — "it won't fail" is not a plan
5. **Prompts SHOULD be testable** — if you can't test it, you can't maintain it
6. **Prompts SHOULD include at least one example** — examples clarify intent better than descriptions
7. **Prompts MUST NOT have conflicting instructions** — contradictions cause unpredictable behavior
8. **Complex prompts SHOULD be split** — if it does 5 things, make 5 prompts

---

## Integration with Bryan's Workflow

This playbook aligns with the workflow defined in `bryan-developer-profile-v1_4.md`:

- **Specs before code** — Prompts are specs for AI behavior; refine them before deploying
- **Zero-trust outputs** — Test prompt outputs; don't assume correctness
- **Small, scoped changes** — Start with narrow prompts; expand only when needed
- **Model-aware** — Tag prompts with intended model tier (Opus for architecture, Sonnet for planning, Codex for execution, Haiku for validation)

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2024-12-31 | Initial playbook: 6 dimensions, 30-question bank, template, invariants |

---

*Prompt Engineer — Clarifying what the agent should do, shouldn't do, and how you'll know it worked.*
