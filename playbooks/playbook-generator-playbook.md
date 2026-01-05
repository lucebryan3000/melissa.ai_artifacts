# Playbook Creator Playbook — Canonical Standard

> **Purpose**: Authoritative, project-wide standard for creating slash commands and companion playbooks using the Thin Command Surface Pattern
> **Version**: 2.6
> **Last Updated**: 2025-12-31

This playbook governs:
- Where logic lives
- How context is controlled
- How token cost is minimized
- How behavior is made deterministic and scalable

This is not guidance. This is a **mandatory architectural contract**.

---

## Command Index

**20 Plays for Creating/Refactoring Playbooks**:

| Play Name | Category | Purpose |
|-----------|----------|---------|
| PLAY: Collect Requirements | Discovery | Convert fuzzy intent into bounded, testable behavior |
| PLAY: Define Plays | Discovery | Create deterministic behavioral map |
| PLAY: Specify Algorithms | Specification | Eliminate interpretation with pseudocode |
| PLAY: Define Invariants | Specification | Prevent drift and accidental reinterpretation |
| PLAY: Define I/O Contracts | Specification | Make parsing and formatting deterministic |
| PLAY: Error Handling Specification | Specification | Define predictable failure behavior |
| PLAY: Test Checklist | Specification | Close specification loopholes |
| PLAY: Performance Targets | Specification | Prevent runaway complexity |
| PLAY: Generate Slash Command | Construction | Create thin dispatch layer |
| PLAY: Update Index File | Construction | Maintain master command registry |
| PLAY: Validate CLAUDE.md Reference | Validation | Ensure CLAUDE.md doesn't waste context |
| PLAY: Quality Validation | Validation | Quantify playbook quality with real measurements |
| PLAY: Verify Installation | Testing | Ensure playbook works in practice |
| PLAY: Echo (Human Review) | Governance | Structured human review checkpoint |
| PLAY: Version Control | Persistence | Ensure traceability and git hygiene |
| PLAY: Benchmark | Measurement | Establish real-world performance metrics |
| PLAY: Error Recovery | Recovery | Safety net for mistakes |
| PLAY: Enhance Existing Playbook | Evolution | Add single play with strict isolation controls |
| PLAY: Refactor Existing Playbook | Evolution | Migrate existing commands to pattern |
| PLAY: Evolve Playbook | Evolution | Continuously improve based on usage |

**Quick Navigation**:
- [Mental Model](#mental-model) - Three-layer architecture
- [Inputs](#inputs-to-the-playbook-creator) - What you need to provide
- [Outputs](#outputs-four-required--one-optional) - What gets created
- [Plays](#plays-used-to-create-playbooks) - The 19 implementation plays
- [Examples](#examples) - See `.claude/commands/playbooks/` for production playbooks
- [Quality Gates](#quality-gates-hard-requirements) - Invariants vs Guardrails

---

## Mental Model: Three-Layer Architecture

Every command follows this strict separation:

```
User → Slash Command → Playbook → Code
       [Dispatch]      [Spec]     [Implementation]
       [Auto-loaded]   [On-demand] [Executed]
```

**Layer Responsibilities**:

| Layer | Role | Auto-loaded | Allowed Content |
|------|-----|------------|-----------------|
| Slash Command | Dispatch + UX | ✅ Yes | Usage, intent, play references |
| Playbook | Executable specification | ❌ No | Algorithms, rules, invariants |
| Code | Runtime behavior | N/A | Actual implementation |

Violation of layer boundaries is a **design error**.

**Directory Layout** (Non-Negotiable):

```
.claude/
├── commands/
│   ├── <command>.md                  ← Slash command (auto-loaded)
│   └── playbooks/
│       ├── .claudeignore             ← MUST exist, contains: *
│       ├── <command>-playbook.md     ← Implementation spec (on-demand)
│       └── playbook-generator-playbook.md  ← This file
├── docs/
│   ├── README-playbooks-and-slash-commands.md  ← Master index (REQUIRED)
│   └── thin-command-surface-pattern.md        ← Pattern reference
└── CLAUDE.md                         ← MUST reference the index file
```

**Token Economy**:
- **Command (auto-loaded)**: Minimal footprint, loaded every session, must be worth the cost
- **Playbook (on-demand)**: Zero baseline cost via `.claudeignore`, unlimited length
- **`.claudeignore` Strategy**: File contains `*`, prevents automatic playbook loading, allows infinite growth at zero cost

**Risk Modes**:
- **sandbox** (default): Experimental, rapid iteration, data loss acceptable, don't overtest during dev
- **elevated**: Shared tools, staging-like, cautious with destructive ops
- **production**: User-facing, real data, robustness over velocity, full testing required

**Simplicity Principle** (Single-dev, MVP/POC work):
- Velocity > Perfect Architecture | Accuracy > Elegance | Working > Complete
- Don't build abstractions for one use case
- Don't overdesign schemas (just enough for current needs)
- Don't add retry logic for errors that never happen
- Build, measure, then optimize (not before)

**Knowledge Discovery** (Typesense Integration):
- Query KB BEFORE creating new playbooks
- Use faceted search (intent, complexity, content_type, authoritative_example)
- Avoid reinventing existing patterns
- See PLAY: Collect Requirements for integration details

---

## Authority Model

The Playbook Creator operates in **dual-mode authority**:

### ENFORCES (Generated Artifacts)

These files are **owned by this playbook** and may be modified directly:
- `.claude/commands/<command>.md`
- `.claude/commands/playbooks/<command>-playbook.md`
- Index entry in `.claude/docs/README-playbooks-and-slash-commands.md` (new entries only)

For these files, the Playbook Creator:
- Makes architectural decisions
- Enforces invariants
- Applies standards directly
- No permission required (within playbook scope)

### VALIDATES + REPORTS (Global/Shared Files)

These files are **owned by the project** and require explicit approval:
- `.claude/CLAUDE.md`
- Existing index content (modifications to existing entries)
- Project-wide configurations

For these files, the Playbook Creator:
- Checks for issues
- Proposes exact fixes
- MUST NOT silently mutate
- Requires explicit approval before changes

**Why This Distinction**:
- Generated artifacts = this playbook's domain
- Global files = project's domain
- Prevents scope creep and silent mutations
- Maintains clear ownership boundaries

---

## Invariants vs Guardrails

### Invariants (MUST / NEVER)

**Definition**: Non-negotiable architectural constraints that define the pattern itself.

**Core Invariants**:
1. **Thin Command Surface** - Slash commands MUST NOT contain algorithms, formulas, or templates
2. **Zero Duplication** - Duplication ratio between command and playbook MUST = 0%
3. **Playbook Isolation** - Playbooks MUST be ignored via `.claudeignore`
4. **Single Source of Truth** - MUST NOT create versioned copies (`<command>-playbook-v2.md`)
5. **Index Maintenance** - MUST update index when creating new commands
6. **CLAUDE.md Indirection** - CLAUDE.md MUST reference index file, NEVER individual playbooks

**Violation = Design Error**

Invariants cannot be bent. If an invariant doesn't fit the use case:
1. STOP
2. Surface the conflict in PLAY: Echo
3. Propose architecture change, OR
4. Conclude this pattern doesn't apply to this use case

---

### Guardrails (SHOULD / STRONGLY RECOMMENDED)

**Definition**: Strong guidance from Bryan Developer Profile and best practices.

**Core Guardrails**:
1. **Small Diffs** - Prefer focused changes over large rewrites
2. **Reuse > Reinvent** - Check existing patterns before creating new
3. **Simple First** - Start with minimal implementation, add complexity only when proven necessary
4. **Test When Done** - Test when feature complete, not during rapid iteration
5. **Token Efficiency** - Command files SHOULD be ≤ 2,000 tokens for prototyping flexibility
6. **Validation-First** - Self-test before reporting completion

**Guardrails CAN be bent**:
- With explicit rationale
- Surfaced in PLAY: Echo deviation summary
- Escalated if bent materially (see severity levels below)

---

### Deviation Severity Levels

**Minor**: Single guardrail bent slightly
- Example: Command file 2,100 tokens instead of ≤2,000
- Action: Note in PLAY: Echo deviation summary, proceed

**Moderate**: Single guardrail bent significantly, OR multiple bent slightly
- Example: Skipped reuse check + command 2,500 tokens
- Action: Surface in PLAY: Echo with rationale, continue if reasoning is sound

**Major**: Multiple guardrails bent significantly, OR core guardrail violated materially
- Example: Large rewrite without spec + skipped testing + 40% duplication
- Action: PAUSE, escalate via PLAY: Echo, ask for explicit confirmation

**Escalation Threshold**: When crossing from Moderate → Major

If deviation crosses into Major territory, Claude MUST pause and ask:
> "This deviates materially from the standard pattern in [X] ways:
> - [Deviation 1 with severity]
> - [Deviation 2 with severity]
>
> Should I proceed with this approach, or would you prefer I redesign?"

---

## Inputs to the Playbook Creator

The creator operates on the following inputs (partial allowed, ambiguity not):

1. **Command Name**
   - `/backlog`, `/harvest`, etc.

2. **Intent**
   - One paragraph, outcome-focused

3. **Plays**
   - Distinct verbs representing behavior

4. **State / Artifacts**
   - Files or memory touched

5. **Constraints**
   - MUST / NEVER rules
   - Persistence rules
   - Context constraints

6. **Examples**
   - Minimum 3 representative invocations

---

## Outputs (Four Required + One Optional)

The creator MUST emit **four required artifacts** and SHOULD emit benchmark data when applicable.

### 1. `<command>-playbook.md`

- Location: `.claude/commands/playbooks/`
- Ignored by default
- Audience: AI agents, implementers
- Purpose: **Executable specification**

**Required Contents**:
- Named plays
- Algorithms and pseudocode
- Decision thresholds
- Invariants
- Error handling
- Testing checklist
- Performance targets

There is no maximum length.

---

### 2. `<command>.md` (Slash Command)

- Location: `.claude/commands/`
- Auto-loaded
- Audience: User
- Purpose: **Dispatch layer**

**Allowed Contents**:
- Usage
- "Runs Play: <Name>"
- When-to-use guidance
- High-level flow summaries
- Design contract

**Forbidden Contents**:
- Algorithms
- Pseudocode
- Thresholds
- Output templates
- Formulas
- Integration logic

---

### 3. README Index Entry

- Location: `.claude/docs/README-playbooks-and-slash-commands.md`
- Purpose: **Master command index**

**Required Update**:
Add entry with:
- Command name and description
- Playbook reference
- Key plays summary
- Token cost estimate

---

### 4. CLAUDE.md Validation

- Location: `.claude/CLAUDE.md`
- Purpose: **Ensure proper referencing**

**Required Check**:
- CLAUDE.md MUST reference: `.claude/docs/README-playbooks-and-slash-commands.md`
- CLAUDE.md MUST NOT directly reference individual playbooks
- If reference missing, auto-suggest addition

---

## Plays Used to Create Playbooks

These plays define how the Playbook Creator operates.

---

### PLAY: Collect Requirements

**Goal**: Convert fuzzy intent into bounded, testable behavior.

**Pre-Flight Check** (Before collecting requirements):

0. **Verify Invariants Applicability**
   - Is this a slash command + playbook pair? (If no, wrong tool)
   - Will this follow Thin Command Surface pattern? (If no, escalate - pattern may not fit)
   - Can logic be separated from dispatch? (If no, reconsider approach)
   - If any invariant conflicts surface, STOP and discuss with human

**Initial Todo List** (REQUIRED - Show at start):

Before beginning the work, you MUST display a clear, concise todo list using the TodoWrite tool showing all major steps:

```markdown
## Creating /<command> Command/Playbook Pair

- [ ] Collect requirements and verify spec quality gates
- [ ] Define plays and create behavioral map
- [ ] Specify algorithms with pseudocode
- [ ] Define invariants and I/O contracts
- [ ] Define error handling specification
- [ ] Create test checklist and performance targets
- [ ] Generate slash command file
- [ ] Update index and validate CLAUDE.md reference
- [ ] Run quality validation
- [ ] Verify installation (self-test)
- [ ] Human review checkpoint (PLAY: Echo)
- [ ] Version control and commit
```

This gives the user visibility into what will happen and allows you to track progress through the session.

**Steps**:
1. Capture command + intent
2. Enumerate verbs → candidate plays
3. Identify state/files touched (which files will playbook reference for context?)
4. Identify invariants and constraints
5. Enumerate edge cases and failures
6. Collect ≥3 examples
7. Check for reusable patterns

**Before Creating New Playbook**:
1. **Query Typesense KB** (if deployed) - See "Knowledge Discovery First" section
   - Search for similar commands/patterns
   - Review authoritative examples
   - Document what exists vs what needs creation
2. Check `.claude/docs/README-playbooks-and-slash-commands.md` for existing similar commands
3. Check `.claude/commands/playbooks/` for reusable patterns
4. Consider: Can we extend existing playbook vs creating new one?
5. If evolution requires new approach: Propose refactoring existing vs creating new

**Reuse > Reinvent** - But evolution may require replacement, not extension.

**Output**: Formal requirements block

**Validation**:
- Intent MUST be outcome-focused (not implementation-focused)
- Plays MUST be verb phrases
- Examples MUST cover happy path + edge cases

**Spec Quality Gates** (MUST be met before proceeding):
- [ ] Purpose is clear (what and why)
- [ ] Inputs and outputs are defined
- [ ] Scope is bounded (in-scope/out-of-scope)
- [ ] Success criteria are testable
- [ ] Dependencies are identified

**If any gate fails**: Spec is incomplete - STOP and refine before generating playbook.

---

### PLAY: Define Plays

**Goal**: Create a deterministic behavioral map.

**Rules**:
- Plays MUST be verb phrases
- Each play MUST define:
  - Trigger
  - Responsibilities
  - Inputs / outputs
  - Invariants

Play names are API contracts and MUST remain stable.

**Validation**:
- No duplicate play names
- All command variants mapped to plays
- Play names consistent between command and playbook

---

### PLAY: Specify Algorithms

**Goal**: Eliminate interpretation.

**Requirements**:
Every non-trivial play MUST include:
- Decision rules
- Thresholds
- Ordering rules
- State mutation rules
- Pseudocode when branching exists

> Prose-only logic is explicitly disallowed for mutating behavior.

**Validation**:
- All conditionals have explicit thresholds
- All loops have termination conditions
- All mutations have before/after state descriptions

---

### PLAY: Define Invariants

**Goal**: Prevent drift and accidental reinterpretation.

**Rules**:
- Use MUST / NEVER / ALWAYS
- Invariants apply globally
- Invariants override play-specific behavior

**Examples**:
- Cleanup MUST run after any mutation
- Slash commands MUST NOT duplicate logic
- Session findings MUST NEVER persist implicitly

**Validation**:
- Invariants section exists
- At least 3 invariants defined
- No contradictory invariants

---

### PLAY: Define I/O Contracts

**Goal**: Make parsing and formatting deterministic.

**Must Define**:
- Canonical file paths
- Stable section headers
- Required metadata (IDs, timestamps)
- Output shape (not templates in command)

**Validation**:
- All file paths absolute or clearly relative
- Section headers stable (won't change)
- Metadata fields required vs optional

---

### PLAY: Error Handling Specification

**Goal**: Predictable failure behavior.

**Must Cover**:
- Missing files
- Malformed formats
- Partial parses
- Dry-run under failure
- Fatal vs non-fatal errors

Error behavior is part of the spec, not an afterthought.

**Validation**:
- Error handling section exists
- All error scenarios have defined behavior
- Dry-run works even on errors

---

### PLAY: Test Checklist

**Goal**: Close specification loopholes.

**Rules**:
- ≥10 checklist items for mutating commands
- Must include:
  - Happy path
  - Edge cases
  - Idempotency
  - Dry-run behavior
  - Failure modes

**Validation**:
- Checklist exists
- Minimum item count met
- All checklist items testable

---

### PLAY: Performance Targets

**Goal**: Prevent runaway complexity.

**Must Specify**:
- Typical-case targets
- Scaling behavior
- Max recommended size (if applicable)

**Validation**:
- Performance section exists
- Targets are measurable
- Scaling limits defined

---

### PLAY: Generate Slash Command

**Goal**: Create the dispatch layer.

**Authoritative Reference**: https://code.claude.com/docs/en/slash-commands

**Required Frontmatter** (MUST be present):

```yaml
---
description: [one-line description, REQUIRED for command discovery]
allowed-tools: [REQUIRED for auto-execution - tools Claude can use]
argument-hint: [optional - show expected args like "[query] [--options]"]
---
```

**Frontmatter Rules**:
- **description**: REQUIRED. Used in `/help` output and enables SlashCommand tool discovery. Keep under 60 chars. Format: "Verb + object"
- **allowed-tools**: REQUIRED for auto-execution. Without this, command will require manual approval for every tool use.
  - Script execution: `Bash(.claude/commands/scriptname.sh:*)`
  - File operations: `[Read, Write, Edit, Glob, Grep]`
  - Multiple tools: `[Read, Write, Edit, Glob, Grep, Bash]`
  - Specific CLI: `Bash(npm test:*), Bash(git:*)`
- **argument-hint**: Optional but recommended. Shows expected syntax in `/help`

**Non-Standard Fields to Remove**:
- `tags` - Not in official Anthropic documentation
- `invoke` - Not in official Anthropic documentation
- `type` - Not in official Anthropic documentation
- `mcp-tool` - Not in official Anthropic documentation

**Slash Command Body MUST**:
- State: "All logic lives in `<playbook>.md`"
- Reference plays by exact name
- Include "Use when" guidance
- End with a Design Contract

**Slash Command Body MUST NOT**:
- Contain algorithms
- Contain formulas
- Duplicate playbook content
- Explain implementation details

**Validation**:
- Frontmatter present with `description` and `allowed-tools` fields
- Minimal context footprint (≤ 2,000 tokens for single-dev prototyping)
- Zero duplication of playbook content
- Design contract present
- All play references match playbook
- No non-standard frontmatter fields

---

### PLAY: Update Index File

**Goal**: Maintain master command registry.

**File**: `.claude/docs/README-playbooks-and-slash-commands.md`

**Required Update**:

```markdown
## Project Commands

### /<command>

**Description**: <One-line description>

**Playbook**: [<command>-playbook.md](.claude/commands/playbooks/<command>-playbook.md)

**Key Plays**:
- Play: <Name> - <Brief description>
- Play: <Name> - <Brief description>

**Token Cost**: <X> tokens (measured via /cost, see PLAY: Benchmark)
**Baseline Cost**: <Y> tokens (old pattern, if converting existing command)

**Usage**:
/<command> <args>  → Play: <Name>
```

**Doc Sprawl Prevention**:

**MUST**:
- Update main playbook in place (no versioned copies)
- Archive obsolete versions in `.claude/commands/playbooks/archive/`
- Keep one canonical source of truth

**MUST NOT**:
- Create `<command>-playbook-v2.md` alongside original
- Duplicate content across multiple playbooks
- Leave abandoned drafts in main directory

**Version History Section** tracks changes, not separate files.

**Validation**:
- Entry added to index
- All links valid
- Token cost calculated
- Alphabetically sorted
- No duplicate/versioned playbooks in main directory

---

### PLAY: Validate CLAUDE.md Reference

**Goal**: Ensure CLAUDE.md doesn't waste context.

**Required Check**:

```markdown
# In CLAUDE.md, MUST have:

## Slash Commands and Playbooks

For available slash commands and playbooks, see:
[.claude/docs/README-playbooks-and-slash-commands.md](.claude/docs/README-playbooks-and-slash-commands.md)

This index provides just enough context without loading full playbooks.
```

**Validation**:
- Reference exists in CLAUDE.md
- No direct playbook references in CLAUDE.md
- No command content duplicated in CLAUDE.md

**Auto-fix** (if missing):
```markdown
Add to CLAUDE.md:

## Slash Commands and Playbooks

For available slash commands and playbooks, see:
[.claude/docs/README-playbooks-and-slash-commands.md](.claude/docs/README-playbooks-and-slash-commands.md)
```

---

### PLAY: Quality Validation

**Goal**: Quantify playbook quality with real measurements, not estimates.

**Metrics to Calculate**:

1. **Duplication Ratio** (Objective, measurable now)
   ```
   duplication = lines_in_both_files / total_lines
   Target: 0%
   ```

2. **Play Coverage** (Objective, measurable now)
   ```
   coverage = mapped_commands / total_commands
   Target: 100%
   ```

3. **Command File Size** (Objective, measurable now)
   ```
   size = command_file_lines
   Target: Record actual, establish baseline over time
   ```

4. **Token Efficiency** (Requires benchmark, see PLAY: Benchmark)
   ```
   MUST be measured via /cost before and after
   DO NOT estimate or guess
   Use PLAY: Benchmark to establish real metrics
   ```

**Quality Gates**:
- Duplication = 0% (measurable now)
- Coverage = 100% (measurable now)
- Size: Record actual (no arbitrary limits without data)
- Efficiency: Measure via benchmark, don't estimate

**Prompt Debt Detection**:

If playbook shows these signs:
- Command repeatedly confuses Claude
- Users restate requirements multiple times
- Plays not executing as intended
- High back-and-forth during execution

**When Detected**:
1. STOP using the failing command/playbook
2. Run quality validation
3. Apply improvements
4. Re-test with refined version

**DO NOT keep reusing a broken playbook.**

**Validation**:
- Duplication ratio calculated (exact)
- Play coverage calculated (exact)
- File size recorded (exact)
- Token efficiency: Benchmark required (see PLAY: Benchmark)
- Report generated with ONLY proven data
- Prompt debt indicators checked

---

### PLAY: Verify Installation

**Goal**: Ensure playbook works in practice.

**Self-Test Protocol** (MANDATORY before reporting completion):

**When to Test**:
- ✅ When you've finished implementing a feature
- ✅ When you're about to say "Perfect! We're done."
- ✅ When you're ready to report completion to the user
- ✅ When nearing completion of a bug fix

**When NOT to Overtest** (Rapid Iteration Mode):
- ❌ During active development/prototyping
- ❌ When making small incremental changes
- ❌ When explicitly in "rapid iteration mode"
- ❌ When caught in a debug loop (see Spin Limit)

**The Trigger**: "I'm done coding → Now I test."

Before reporting completion to the user, you MUST:

1. **Command Loading**
   ```bash
   # Start new session
   claude

   # Verify command appears
   /help | grep <command>
   ```

2. **Playbook Excluded**
   ```bash
   # In conversation
   /context
   # Verify playbook NOT in loaded files
   ```

3. **Play References Work**
   ```bash
   # Run command
   /<command> <args>
   # Should work without errors
   ```

4. **Test All Plays/Variations**
   - Run each play at least once
   - Test error cases
   - Verify outputs match spec

5. **On-Demand Loading**
   ```bash
   # In conversation
   "Show me the <command> playbook details"
   # Claude should Read the playbook file
   ```

6. **Token Cost Baseline**
   ```bash
   /cost
   # Record actual token usage (do NOT compare to estimates)
   # Use for PLAY: Benchmark if needed
   ```

**Spin Limit Rule** (Hard Constraint):
- Max 2 attempts per debugging approach
- If same strategy fails twice → STOP
- Explain why it failed
- Propose new plan (smaller tasks, new spec, or focused question)
- NEVER repeat a failing approach more than twice

**If stuck in debug loop**:
- STOP testing
- Report current state
- Ask for guidance or pivot approach

**Validation**:
- All tests pass
- No errors during execution
- Actual token usage recorded (not compared to guesses)
- Ready for user acceptance testing (UAT)

**User Testing (UAT) is for**:
- Human experience, UX feel
- Visual design
- Domain-specific edge cases

**NOT for finding bugs automated tests should catch.**

---

### PLAY: Echo (Human Review)

**Goal**: Structured human review checkpoint before version control.

**When to Run**: After PLAY: Verify Installation completes successfully, before PLAY: Version Control.

**Critical Timing**: Run this when you're about to say **"Code is done, feature is complete."**

This is the handoff moment: Implementation → Human Review → Version Control

**This is NOT**:
- Asking permission for every decision during development
- Blocking rapid iteration
- A bureaucratic form to fill out
- Required during active development/prototyping

**This IS**:
- Thinking out loud about uncertainty
- Surfacing deviations from standard patterns
- Asking questions Claude cannot confidently answer
- Ensuring human stays meaningfully engaged
- Final quality gate before git commit

**Philosophy**: Governance without bureaucracy.

---

### Output Structure (Fixed Sections, Adaptive Voice)

PLAY: Echo MUST produce exactly these 3 sections (in order):

#### 1. Summary (2-4 sentences)

Brief overview of what was created:
- Command name and purpose
- Core pattern used (Thin Command Surface)
- Number of plays defined
- Key architectural decisions

**Tone**: Concise, factual, no marketing language

---

#### 2. Deviation Summary

**If no deviations**:
```
No deviations from standard pattern.
```

**If deviations exist**, list each with:
- **Severity**: Minor / Moderate / Major
- **What**: Specific guardrail bent
- **Why**: Rationale for deviation
- **Impact**: What this means for maintenance/evolution

**Example**:
```
Deviations from guardrails:

1. **Minor** - Command file size: 2,100 tokens (target: ≤2,000)
   - Rationale: Complex argument parsing required for 5 play variations
   - Impact: Slightly higher baseline cost, still acceptable

2. **Moderate** - Skipped Typesense KB discovery
   - Rationale: Typesense not yet deployed in this project
   - Impact: May duplicate existing patterns, recommend KB check later
```

**Escalation Check**:

If deviation crosses into **Major** territory, Claude MUST pause and ask:
```
⚠️ ESCALATION REQUIRED

This deviates materially from the standard pattern in [X] ways:
- [Deviation 1 with severity and impact]
- [Deviation 2 with severity and impact]

This may indicate:
- Pattern mismatch (Thin Command Surface doesn't fit this use case)
- Scope creep (trying to do too much in one command)
- Design issue (needs rethinking)

Should I proceed with this approach, or would you prefer I redesign?
```

---

#### 3. 5 Review Questions (Exactly 5, Always)

**Question Categories** (Fixed, Wording Adaptive):

Each question must come from one of these 5 categories. Wording should be specific to the command being reviewed, not generic.

**Category 1: Intent Alignment**
- Did I correctly understand your actual workflow?
- Does this command match how you'll really use it?
- Is the granularity right (too specific vs too generic)?

**Category 2: Structural Correctness**
- Is the play decomposition logical and maintainable?
- Are the boundaries between plays clear enough?
- Did I split or combine plays appropriately?

**Category 3: Missing Behaviors / Edge Cases**
- What uncommon scenario did I likely miss?
- What happens when [specific edge case]?
- Are there failure modes I didn't account for?

**Category 4: Specification Quality**
- Is the error handling comprehensive enough?
- Are the invariants too strict or too loose?
- Did I over-specify or under-specify anything?
- Should the playbook reference any additional files for context (config, templates, project docs)?

**Category 5: Integration Impact**
- How will this interact with your existing commands?
- What project context am I missing?
- Are there dependencies I should be aware of?

**Question Design Rules**:
- ✅ Ask questions Claude CANNOT answer confidently
- ✅ Specific to this command (not generic)
- ✅ Reveal actual uncertainty or assumptions
- ❌ Questions with obvious answers
- ❌ Generic questions ("Does this look good?")
- ❌ Rubber-stamping ("Everything perfect?")

**Example Questions** (for `/backlog` command):
1. **Intent**: Does the automatic cleanup on every add match your workflow, or would you prefer manual cleanup control?
2. **Structure**: Should deduplication be a separate play, or keep it embedded in cleanup?
3. **Edge Cases**: What should happen if BACKLOG.md gets corrupted mid-session?
4. **Quality**: The playbook references `_build/prompts/BACKLOG.md` for state - should it reference any other files for context (config, templates, project docs)?
5. **Integration**: How does this interact with your existing `/harvest` command - any overlap or conflicts?

---

### Question Bank (Hybrid Approach)

**Reference**: For detailed question templates and examples, see [playbook-creation-question-bank.md](playbook-creation-question-bank.md).

**5 Fixed Categories** (must cover all 5):
1. Intent Alignment
2. Structural Correctness
3. Missing Behaviors / Edge Cases
4. Specification Quality
5. Integration Impact

**Using the Question Bank**:
1. Read `playbook-creation-question-bank.md` before writing PLAY: Echo questions
2. Select 1 question template from each of the 5 categories
3. Adapt wording to reference specific plays, files, and features from the current command
4. Verify questions reveal genuine uncertainty (not rubber-stamping)

**Anti-Patterns**:
- ❌ Asking questions you CAN answer confidently
- ❌ All 5 questions from same category
- ❌ Generic questions that apply to any command
- ❌ Leading questions that telegraph "correct" answer
- ❌ Permission-seeking on architectural invariants (those are enforced, not negotiable)

---

### Complete PLAY: Echo Output Example

```markdown
## Human Review Checkpoint

### Summary
Created `/backlog` command with Thin Command Surface pattern. 5 plays defined: Add Note, Status Report, List/Review, Manual Update, Sync Session Findings. Automatic cleanup runs on every add operation. Command file: 125 lines (~650 tokens).

### Deviation Summary
No deviations from standard pattern.

### Review Questions

1. **Intent Alignment**: Does automatic cleanup on every add match your workflow, or would you prefer explicit cleanup control?

2. **Structural Correctness**: Should deduplication be a separate play (callable independently), or is embedding it in cleanup the right approach?

3. **Missing Behaviors**: What should happen if BACKLOG.md gets corrupted or has malformed YAML during a session?

4. **Specification Quality**: The health score formula uses staleness + density + duplication - are there other backlog quality indicators I'm missing?

5. **Integration Impact**: How does `/backlog` interact with your `/harvest` command - any workflow overlap or potential conflicts I should address?

---

**Awaiting human confirmation before proceeding to version control.**
```

---

### Validation Checklist

Before completing PLAY: Echo:
- [ ] Summary is 2-4 sentences (not a wall of text)
- [ ] Deviation summary exists (even if "none")
- [ ] Exactly 5 questions produced
- [ ] All 5 categories covered
- [ ] Questions reveal actual uncertainty (not rubber-stamping)
- [ ] Questions are specific to this command (not generic)
- [ ] Escalation check performed (if Major deviations)
- [ ] Ends with "Awaiting human confirmation" state

---

### Integration with Workflow

**Standard Flow**:
```
PLAY: Verify Installation (self-test)
    ↓
PLAY: Echo (human review) ← YOU ARE HERE
    ↓
Human responds
    ↓
PLAY: Version Control (if approved)
```

**If Human Requests Changes**:
```
PLAY: Echo surfaces issues
    ↓
Human: "Please change X"
    ↓
Make changes
    ↓
PLAY: Verify Installation (re-test)
    ↓
PLAY: Echo (brief re-review)
    ↓
PLAY: Version Control
```

---

**When NOT to Run PLAY: Echo**:
- During active development/prototyping
- When iterating on playbook in rapid cycles
- When explicitly in "bypass review" mode
- For trivial changes (typo fixes, formatting)

**When MUST Run PLAY: Echo**:
- Creating new command/playbook pair
- Refactoring existing command to new pattern
- Making significant changes to existing playbook
- Before claiming "implementation complete"

---

### PLAY: Version Control

**Goal**: Ensure traceability and git hygiene.

**Required Actions**:

1. **Commit Together**
   ```bash
   git add .claude/commands/<command>.md
   git add .claude/commands/playbooks/<command>-playbook.md
   git add .claude/docs/README-playbooks-and-slash-commands.md
   git add .claude/CLAUDE.md  # if updated
   ```

2. **Conventional Commit**
   ```
   feat(commands): add /<command> with thin command surface pattern

   - Slash command: .claude/commands/<command>.md (650 tokens)
   - Playbook: .claude/commands/playbooks/<command>-playbook.md (on-demand)
   - Token savings: XX% (YY tokens)
   - Plays: <list plays>

   Updated master index and CLAUDE.md reference.
   ```

3. **Tag if Applicable**
   ```bash
   git tag -a v<command>-1.0 -m "Initial release of /<command>"
   ```

**Validation**:
- Files committed together
- Commit message follows convention
- Commit message contains ONLY measured data (no estimates)
- Benchmark results included if claiming efficiency
- CHANGELOG updated if exists

---

### PLAY: Benchmark

**Goal**: Establish real-world performance metrics through actual measurement.

**When to Run**:
- After creating new command/playbook pair
- When claiming token savings
- Before documenting efficiency improvements
- Never estimate - always measure

**Benchmark Procedure**:

1. **Baseline Measurement (Before Pattern)**
   ```bash
   # If converting existing command:
   # Start fresh session
   claude

   # Run /cost to get baseline
   /cost
   # Record: input_tokens, output_tokens, total_cost

   # Run the command that loads old implementation
   /<command>

   # Check /cost again
   /cost
   # Record: tokens_after_command

   # Calculate: baseline_cost = tokens_after_command - initial_tokens
   ```

2. **Pattern Measurement (After Thin Command Surface)**
   ```bash
   # Start fresh session
   claude

   # Run /cost to get baseline
   /cost
   # Record: input_tokens, output_tokens

   # Run new thin command
   /<command>

   # Check /cost
   /cost
   # Record: tokens_after_thin_command

   # Calculate: thin_cost = tokens_after_thin_command - initial_tokens
   ```

3. **On-Demand Playbook Load Test**
   ```bash
   # In same session, explicitly reference playbook
   "Show me the implementation details from the playbook"

   # Claude should Read playbook file
   # Check /cost
   /cost
   # Record: tokens_after_playbook_load

   # Calculate: playbook_load_cost = tokens_after_playbook_load - tokens_after_thin_command
   ```

4. **Calculate Real Efficiency**
   ```bash
   # ONLY now can you calculate:
   token_reduction = baseline_cost - thin_cost
   efficiency_percent = (token_reduction / baseline_cost) * 100

   # If playbook was loaded:
   total_with_playbook = thin_cost + playbook_load_cost
   net_savings = baseline_cost - total_with_playbook
   net_efficiency = (net_savings / baseline_cost) * 100
   ```

5. **Document Proven Metrics**
   ```markdown
   ## Benchmark Results - /<command>

   **Date**: YYYY-MM-DD
   **Claude Version**: X.Y.Z
   **Model**: claude-sonnet-4.5

   ### Before (Old Pattern)
   - Baseline cost: X tokens
   - Command file size: Y lines

   ### After (Thin Command Surface)
   - Thin command cost: A tokens
   - Command file size: B lines
   - Playbook size: C lines (0 tokens until loaded)
   - Playbook load cost: D tokens (when referenced)

   ### Proven Savings
   - Token reduction: X - A = Z tokens
   - Efficiency: (Z / X) * 100 = W%
   - File size reduction: (Y - B) / Y * 100 = V%

   ### Net Impact (if playbook loaded)
   - Total cost with playbook: A + D = E tokens
   - Net savings: X - E = F tokens
   - Net efficiency: (F / X) * 100 = G%

   ### Conclusion
   - Immediate savings: W% (Z tokens)
   - Breakeven point: [Playbook loaded or not]
   - Recommendation: [Based on actual data]
   ```

**Anti-Patterns (NEVER Do This)**:
- ❌ Estimating "50-60% savings" without measurement
- ❌ Guessing token costs
- ❌ Assuming efficiency without /cost data
- ❌ Extrapolating from one command to all commands
- ❌ Using line count as proxy for tokens (ratio varies)

**Validation**:
- /cost output captured at each step
- All calculations shown with source data
- No estimates or assumptions in metrics
- Benchmark results committed to repo
- Future claims reference this benchmark

---

### PLAY: Error Recovery

**Goal**: Safety net for mistakes.

**Spin Limit Rule** (Hard Constraint):
- Max 2 attempts per approach
- If same strategy fails twice → STOP
- Explain why it failed
- Propose new plan (smaller tasks, new spec, or focused question)
- NEVER repeat a failing approach more than twice

**When to Trigger Error Recovery**:
- Hit spin limit (2 failed attempts with same approach)
- Command fundamentally broken
- Playbook has critical errors
- Actual token usage unexpected (not "higher than estimated" - we don't estimate)

**Rollback Procedure**:

1. **Identify Problem**
   - Command not working
   - Playbook has errors
   - Hit spin limit with current approach
   - Actual token usage unexpected

2. **Locate Previous Version**
   ```bash
   git log --follow .claude/commands/<command>.md
   git show <commit>:.claude/commands/<command>.md
   ```

3. **Revert to Working Version**
   ```bash
   git revert <commit>
   # Or
   git checkout <commit> -- .claude/commands/<command>.md
   git checkout <commit> -- .claude/commands/playbooks/<command>-playbook.md
   ```

4. **Archive Broken Version**
   ```bash
   mkdir -p .claude/commands/archive
   mv .claude/commands/<command>.md \
      .claude/commands/archive/<command>-broken-$(date +%Y%m%d).md
   ```

5. **Update Index**
   - Remove broken entry from README
   - Document what went wrong

**Validation**:
- Rollback procedure documented
- Archive directory exists
- Index remains consistent
- New approach proposed (not repeating failed strategy)

---

### PLAY: Enhance Existing Playbook

**Goal**: Add a single new play to an existing playbook with strict isolation controls. Zero changes to other plays or command file.

**Inputs**:
1. Existing playbook file (e.g., `.claude/commands/playbooks/<command>-playbook.md`)
2. New play name, algorithm, and test criteria

**When to Use**:
- Adding one capability to a working playbook
- Extending functionality without refactoring
- Minimal risk, surgical addition of functionality
- No changes to existing plays or the command file

**Initial Todo List** (REQUIRED - Show at start):

Before beginning the enhancement, you MUST display a clear todo list using the TodoWrite tool:

```markdown
## Enhancing /<command> Playbook (Adding New Play)

- [ ] Validate scope and check for play duplication
- [ ] Define new play with complete specification
- [ ] Create test checklist (≥3 test cases)
- [ ] Run isolation check (no changes to existing plays/command)
- [ ] Insert play into playbook at appropriate location
- [ ] Self-test all test cases
- [ ] Report results (playbook modified, command unchanged)
```

This gives visibility into the surgical procedure and ensures all isolation controls are followed.

**Constraints (NON-NEGOTIABLE)**:
- MUST NOT modify any existing plays
- MUST NOT change the slash command file (`.claude/commands/<command>.md`)
- MUST NOT reorganize playbook structure
- MUST NOT update index or version history (user updates manually if needed)
- MUST NOT touch the command file, period
- Only adds one new play section with complete specification

**Enhancement Procedure**:

1. **Validate Scope**
   - Confirm existing playbook exists and is well-formed
   - Verify new play doesn't duplicate existing plays
   - Check that new play is truly additive (extends, doesn't refactor)
   - If existing plays need changes → recommend PLAY: Refactor instead

2. **Define New Play**

   Provide complete specification for the new play:
   - **Play Name** (unique within playbook)
   - **Goal**: One-sentence purpose
   - **Trigger Condition**: When this play runs
   - **Algorithm**: Pseudocode or step-by-step procedure
   - **Inputs**: Parameters, preconditions
   - **Outputs**: Return values, side effects
   - **Invariants**: MUST/NEVER rules specific to this play
   - **Error Cases**: Failure modes and recovery

3. **Test Checklist**

   Define measurable test criteria:
   ```
   Test Case 1: [Scenario] → Expected: [Result]
   Test Case 2: [Scenario] → Expected: [Result]
   Test Case 3: [Error case] → Expected: [Error handling]
   ```

   Validation:
   - ≥3 test cases defined
   - All cases are measurable/executable
   - Both happy path and error cases covered
   - Tests focus on new play only (don't retest existing plays)

4. **Isolation Check**

   Before adding, verify:
   - No line changes to existing plays ✓
   - No changes to command file ✓
   - No reorganization of playbook sections ✓
   - New play references only existing data structures (no new concepts) ✓
   - Playbook remains self-contained ✓

5. **Insert Play**

   Find the appropriate insertion point:
   - Group by logical category (all retrieval plays together, all editing plays together)
   - Insert before "Explicit Non-Goals" section
   - Use same play template format as existing plays:

   ```markdown
   ### PLAY: <New Play Name>

   **Goal**: <One-sentence purpose>

   **Trigger**: <When this play runs>

   **Algorithm**:
   1. <Step>
   2. <Step>
   3. ...

   **Inputs**:
   - <Parameter>: <Type> - <Description>

   **Outputs**:
   - <Return value>: <Type> - <Description>

   **Invariants**:
   - MUST <rule>
   - NEVER <rule>

   **Error Handling**:
   - If <condition> → <recovery>

   **Test Checklist**:
   - [ ] Test Case 1: <Scenario> → <Expected>
   - [ ] Test Case 2: <Scenario> → <Expected>
   - [ ] Test Case 3: <Scenario> → <Expected>
   ```

6. **Self-Test (Before Reporting)**

   Execute all test cases manually:
   - Can new play be called independently?
   - Does it produce expected outputs?
   - Do error cases handle gracefully?
   - Does it work without affecting existing plays?
   - No side effects on other plays?

   Validation:
   - All test cases pass ✓
   - No regression in existing plays ✓
   - New play is fully isolated ✓
   - Playbook structure unchanged ✓

**Output**:
- Modified playbook with new play inserted
- All test cases in checklist format
- Self-test results confirming isolation
- No changes to command file
- No changes to index (user decides if needed)

**Quality Gates**:
- Zero changes to existing plays
- Zero changes to command file
- Zero structural reorganization
- ≥3 test cases defined and passing
- Complete algorithm specification
- Isolation verified

**Example**:

If playbook has plays:
- PLAY: Retrieve Article
- PLAY: Search Index
- PLAY: List All

You can add:
- ✅ PLAY: Export Results (new capability)
- ✅ PLAY: Filter by Date (new capability)

You CANNOT:
- ❌ PLAY: Search Index (refactor existing)
- ❌ PLAY: Retrieve Article v2 (duplicate)
- ❌ Reorganize section order

---

### PLAY: Refactor Existing Playbook

**Goal**: Migrate existing command/playbook pair to follow Thin Command Surface Pattern while preserving all content.

**Inputs**:
1. Existing slash command file (e.g., `.claude/commands/<command>.md`)
2. Existing playbook file (e.g., `.claude/commands/playbooks/<command>-playbook.md`)

**When to Use**:
- Converting old-style commands to thin surface pattern
- Cleaning up commands with duplicated logic
- Standardizing playbook structure
- Applying new standards to legacy commands

**Initial Todo List** (REQUIRED - Show at start):

Before beginning the refactoring, you MUST display a clear todo list using the TodoWrite tool:

```markdown
## Refactoring /<command> to Thin Command Surface Pattern

- [ ] Analyze current state (read both files, calculate duplication ratio)
- [ ] Extract algorithmic content from command file
- [ ] Reorganize playbook to canonical structure (11 sections)
- [ ] Recreate slash command as thin dispatch layer
- [ ] Update index entry with before/after metrics
- [ ] Validate refactoring (quality checks + content preservation)
- [ ] Run PLAY: Verify Installation (self-test)
- [ ] Optional: Run PLAY: Benchmark (if claiming efficiency)
```

This gives visibility into the refactoring process and ensures nothing is lost.

**Refactoring Procedure**:

1. **Analyze Current State**
   - Read slash command file completely
   - Read playbook file completely
   - Identify content in command that belongs in playbook:
     - Algorithms and pseudocode
     - Output templates
     - Formulas and calculations
     - Integration details
     - Error handling details
     - Performance specifications
   - Calculate current duplication ratio:
     ```
     duplication = lines_appearing_in_both / total_lines
     ```
   - Record current file sizes (command lines, playbook lines)
   - Record current token costs via /cost (optional but recommended)

2. **Extract Content from Command**
   - Identify all plays currently defined
   - Extract all algorithmic content (loops, conditionals, transformations)
   - Extract all output templates (example outputs, formats)
   - Extract all formulas/thresholds (calculations, magic numbers)
   - Extract all integration logic (API calls, file operations)
   - Extract all error handling details (recovery procedures, validation)
   - Keep only in command:
     - Play references ("Runs Play: <Name>", "→ Play: <Name>")
     - "Use when" guidance
     - High-level flow summaries
     - Design contract
     - Frontmatter (description, tags)

3. **Reorganize Playbook**

   Follow canonical structure (see Canonical Playbook Structure section):

   1. **Title + Purpose** (with version and date)
   2. **Mental Model** (add if missing)
   3. **Inputs / Outputs** (formalize if informal)
   4. **Canonical Sections / Entities** (file formats, data structures)
   5. **Plays** (with complete algorithms from extracted content)
   6. **I/O Contracts** (file paths, section headers, metadata)
   7. **Invariants** (MUST/NEVER rules)
   8. **Error Handling** (from extracted content)
   9. **Testing Checklist** (formalize existing examples)
   10. **Performance Considerations** (from extracted content)
   11. **Explicit Non-Goals** (add if missing)

   **Integration Steps**:
   - Move extracted algorithms into appropriate plays
   - Convert prose-only descriptions to pseudocode
   - Add missing sections (Mental Model, Invariants, etc.)
   - Ensure all plays have:
     - Trigger condition
     - Responsibilities
     - Inputs/outputs
     - Invariants
     - Algorithms (if non-trivial)
   - Add version history entry
   - Update last modified date

4. **Recreate Slash Command**

   Use thin command template:

   ```markdown
   ---
   description: <One-line description>
   tags: [<category>, <type>]
   ---

   # /<command> — <Title>

   <One-paragraph purpose>

   All logic lives in `<command>-playbook.md`.
   This file is a thin, user-facing command surface.

   ---

   ## Usage

   /<command> <args>  → Play: <Name>
   /<command> <flag>  → Play: <Name>

   --dry-run         → Preview any play

   ---

   ## <Section for each major usage pattern>

   ### When to use
   - <Scenario 1>
   - <Scenario 2>

   ### Runs
   - Play: <Name>
   - Play: <Name>

   ---

   ## Design Contract

   - This file stays lightweight
   - Logic is never duplicated here
   - Plays are referenced, not redefined
   - `.claudeignore` should exclude playbooks, not commands
   ```

   **Target**: Minimal context footprint (aim for <1,000 tokens for single-dev prototyping)

5. **Update Index**

   Update entry in `.claude/docs/README-playbooks-and-slash-commands.md`:

   ```markdown
   ### /<command>

   **Description**: <One-line description>

   **Playbook**: [<command>-playbook.md](.claude/commands/playbooks/<command>-playbook.md)

   **Key Plays**:
   - Play: <Name> - <Brief description>
   - Play: <Name> - <Brief description>

   **Token Cost**: <X> tokens (measured via /cost, see PLAY: Benchmark)
   **Baseline Cost**: <Y> tokens (old pattern, measured via /cost)

   **Usage**:
   /<command> <args>  → Play: <Name>
   ```

   Note: Record file sizes and run PLAY: Benchmark to establish real metrics

6. **Validate Refactoring**

   **Quality Checks**:
   - [ ] Duplication ratio = 0% (no content in both files)
   - [ ] Play coverage = 100% (all commands mapped to plays)
   - [ ] Command file has minimal footprint (≤ 2,000 tokens for prototyping flexibility)
   - [ ] All play names consistent between files
   - [ ] Design contract present in command
   - [ ] Playbook follows canonical structure (11 sections)
   - [ ] All algorithms formalized (pseudocode, not prose-only)
   - [ ] Error handling comprehensive
   - [ ] Testing checklist exists
   - [ ] Index entry updated

   **Content Preservation Check**:
   - [ ] All original content present in one file or the other
   - [ ] No information lost during refactoring
   - [ ] All examples preserved or improved
   - [ ] All edge cases documented

7. **Run PLAY: Verify Installation**

   ```bash
   # Start fresh session
   claude

   # Verify command appears
   /help | grep <command>

   # Check context (playbook should NOT be loaded)
   /context

   # Test command execution
   /<command> <args>

   # Test on-demand playbook loading
   "Show me the <command> playbook details"

   # Record actual token usage
   /cost
   ```

8. **Optional: Run PLAY: Benchmark**

   If claiming efficiency improvements:
   - Measure old pattern cost (if still available)
   - Measure thin command cost
   - Measure playbook load cost
   - Calculate proven savings
   - Document in benchmark file

   See PLAY: Benchmark for detailed procedure.

**Output**:
- Refactored playbook file (canonical structure, complete)
- Refactored command file (thin surface, 500-700 tokens)
- Updated index entry
- Before/after metrics:
  - Old command size: X lines
  - New command size: Y lines
  - Duplication ratio: old % → 0%
  - (Optional) Token costs from PLAY: Benchmark

**Validation**:
- All content preserved (nothing lost)
- Zero duplication between files
- Playbook follows canonical structure (11 sections)
- Command follows thin surface pattern
- All quality gates passed (see step 6)
- Command works in practice (verified via step 7)

**Anti-Patterns**:
- ❌ Deleting content instead of moving it
- ❌ Leaving algorithms in command file
- ❌ Leaving output templates in command file
- ❌ Skipping sections in canonical structure
- ❌ Not validating content preservation
- ❌ Estimating improvements without measuring

---

### PLAY: Evolve Playbook

**Goal**: Continuously improve playbook based on real usage patterns.

**When to Run**: After executing any play from this playbook.

**Evolution Recommendations**:

After completing a play, analyze the session context and suggest:

1. **Missing Process Steps**
   - Identify gaps in current workflow
   - Propose new procedural steps
   - Suggest checklist additions

2. **New Test Criteria**
   - What edge cases emerged during execution?
   - What validation would have caught issues earlier?
   - What success criteria should be added?

3. **Process Improvements**
   - What steps were unclear or ambiguous?
   - What required back-and-forth that could be streamlined?
   - What decisions needed clarification?

**Output Format**:

After completing a play, provide:

```markdown
## Playbook Evolution Recommendations

**Based on this session, consider adding**:

1. [Specific improvement #1]
2. [Specific improvement #2]
3. [Specific improvement #3]

**Suggested additions to playbook**:
- Process: [New step or clarification]
- Checklist: [New validation item]
- Test criteria: [New success criterion]

---

## Next Steps (Based on Recent Context)

Recommended next actions:

1. **[Action 1]** - [Brief rationale]
2. **[Action 2]** - [Brief rationale]
3. **[Action 3]** - [Brief rationale]
4. **[Action 4]** - [Brief rationale]
5. **[Action 5]** - [Brief rationale]

**Prioritization**: [Which of these 5 is most critical and why]
```

**Integration**:
- Recommendations are suggestions, not automatic changes
- User decides which improvements to incorporate
- Version history tracks evolution over time
- Keep playbook as living document that improves with use

**Anti-Patterns**:
- ❌ Auto-modifying playbook without user approval
- ❌ Suggesting generic improvements not tied to actual session
- ❌ Overwhelming with too many suggestions (max 5)
- ❌ Not explaining rationale for recommendations

---

## Canonical Playbook Structure (Required Order)

1. Title + Purpose (with version and date)
2. Mental Model
3. Inputs / Outputs
4. Canonical Sections / Entities
5. Plays (with algorithms)
6. I/O Contracts
7. Invariants
8. Error Handling
9. Testing Checklist
10. Performance Considerations
11. Explicit Non-Goals

Deviation is a spec violation.

---

## Quality Gates (Hard Requirements)

A playbook is considered valid ONLY if all invariants are met. Guardrails should be followed but deviations are tracked via PLAY: Echo.

### Invariants (MUST - Zero Tolerance)

**Architectural Constraints (Cannot Be Bent)**:
- [x] Slash command is thin and logic-free (no algorithms, formulas, templates)
- [x] Duplication ratio = 0% (measured)
- [x] Playbook resides in `.claude/commands/playbooks/`
- [x] `.claudeignore` exists and ignores playbooks
- [x] Index file updated with new command entry
- [x] CLAUDE.md references index file (not individual playbooks)
- [x] CLAUDE.md does NOT reference playbooks directly
- [x] Single source of truth (no versioned copies: `<command>-playbook-v2.md`)

**Verification (Must Pass)**:
- [x] Command loads in `/help`
- [x] Playbook excluded from `/context`
- [x] On-demand loading works
- [x] All plays execute without errors

---

### Guardrails (SHOULD - Track Deviations)

**Quality Standards (Can Be Bent with Rationale)**:
- [ ] All plays named and referenced consistently
- [ ] All non-trivial logic includes algorithms or pseudocode
- [ ] Invariants (playbook-specific) are explicit and global
- [ ] Error handling is comprehensive
- [ ] Testing checklist exists
- [ ] Command file ≤ 2,000 tokens (prototyping flexibility)
- [ ] Play coverage = 100% (measured)
- [ ] Token efficiency measured via PLAY: Benchmark (when claiming savings)
- [ ] All links in index are valid
- [ ] Files committed together with conventional commit message
- [ ] CHANGELOG updated (if exists)
- [ ] Reuse checked before creating new pattern
- [ ] Typesense KB queried (if deployed)
- [ ] Self-tested before PLAY: Echo

**Deviation Handling**:
- Guardrail deviations are surfaced in PLAY: Echo
- Classified as Minor / Moderate / Major
- Rationale required for all deviations
- Major deviations trigger escalation

---

## Examples

**For complete production examples, see**:
- [`.claude/commands/playbooks/`](.claude/commands/playbooks/) - All production playbooks with full implementations

**Current Production Playbooks**:
1. `backlog-playbook.md` - Smart backlog management (771 lines, comprehensive example)
2. `haiku-chunk-navigation-playbook.md` - KB chunk compilation workflow
3. `kbgen-validate-playbook.md` - KB validation procedures
4. `lean-skill-playbook.md` - Lean skill creation pattern
5. `mcp-prompt-create-playbook.md` - MCP/ChatGPT prompt templates

Each demonstrates:
- Complete Thin Command Surface pattern
- Canonical playbook structure (11 sections)
- Zero duplication between command and playbook
- On-demand loading via `.claudeignore`
- Full algorithm specifications
- Comprehensive error handling
- Testing checklists

**Minimal Inline Example** (Creating `/hello` Command):

```markdown
# Input Requirements
Command: /hello
Intent: Greet user with timestamp
Plays: Greet User
State: None (stateless)
Constraints: MUST include UTC timestamp

# Output: .claude/commands/hello.md (thin command)
---
description: Greet user with timestamp
---
# /hello
All logic lives in `hello-playbook.md`.

/hello  → Play: Greet User

# Output: .claude/commands/playbooks/hello-playbook.md (playbook)
## PLAY: Greet User
**Algorithm**:
1. Get current UTC timestamp
2. Format: "Hello! Current time: YYYY-MM-DD HH:MM:SS UTC"
3. Output formatted greeting
```

**For detailed before/after comparisons, benchmark data, and complex multi-play examples**: Review the production playbooks listed above

---

## Reference Documentation

### Pattern Documentation
- [thin-command-surface-pattern.md](.claude/docs/thin-command-surface-pattern.md) - Official pattern specification

### Master Index
- [README-playbooks-and-slash-commands.md](.claude/docs/README-playbooks-and-slash-commands.md) - Master command registry

### Project Standards
- [CLAUDE.md](.claude/CLAUDE.md) - Project-specific instructions

---

## Explicit Non-Goals

- This playbook does not write code
- This playbook does not allow reference-only specs
- This playbook does not permit logic duplication
- This playbook does not optimize for brevity over determinism
- This playbook does not create playbooks without updating the index
- This playbook does not allow CLAUDE.md to reference playbooks directly

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 2.6 | 2025-12-31 | **Initial Todo List Requirement**: Added mandatory todo list display at start of PLAY: Collect Requirements, PLAY: Enhance Existing Playbook, and PLAY: Refactor Existing Playbook. Uses TodoWrite tool to give users immediate visibility into workflow steps and enable progress tracking. Improves transparency and session management. |
| 2.5 | 2025-12-31 | **PLAY: Enhance Existing Playbook**: Added surgical play-addition capability with strict isolation controls. Enables adding single plays without refactoring entire playbook. Includes validation, test checklist, self-test protocol, and strict constraints (no existing play modifications, no command file changes, no reorganization). Updated Command Index (19 → 20 plays). |
| 2.4 | 2025-12-30 | **Simplification**: Added Command Index table (19 plays with categories), consolidated Mental Model sections (10 sections → 1 unified section, ~200 lines saved), replaced Examples section with reference to production playbooks in `.claude/commands/playbooks/` (~130 lines saved). Total reduction: ~330 lines (17%). Extracted question bank to separate file (v2.3). |
| 2.3 | 2025-12-30 | **Governance Framework**: Added PLAY: Echo (human review checkpoint with 5-question framework), Authority Model (dual-mode: enforce vs validate+report), Invariants vs Guardrails distinction (MUST vs SHOULD), Deviation Tracking (Minor/Moderate/Major severity), Escalation thresholds. Operationalized "governance without bureaucracy". Updated Quality Gates to separate invariants (zero tolerance) from guardrails (track deviations). |
| 2.2 | 2025-12-30 | Added Simplicity Principle (velocity > architecture for MVP/POC work) and Knowledge Discovery First (Typesense integration for finding existing patterns before creating new ones). |
| 2.1 | 2025-12-30 | Added Bryan profile principles: Spec Quality Gates, Spin Limit Rule, Token Economy, Risk Modes, 80/20 Config, Self-Test Protocol, Prompt Debt Detection, Doc Sprawl Prevention, Playbook Evolution recommendations. Doubled token flexibility for prototyping (≤2,000 tokens). |
| 2.0 | 2025-12-30 | Added index update, CLAUDE.md validation, quality metrics, examples, Refactor play, Benchmark play |
| 1.0 | 2025-12-26 | Initial canonical standard |

---

**Maintained by**: Project maintainers
**Questions**: File issue or update this playbook
**Status**: Production standard, apply to all new commands
