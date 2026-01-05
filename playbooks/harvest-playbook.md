# HARVEST PLAYBOOK

Comprehensive post-implementation review workflow. All plays referenced in `/harvest` command.

---

## PLAY: Context Snapshot

**Purpose**: Establish full context of what was just implemented before running detection.

**Input**: User provides implementation description (optional; you can infer from git)

**Steps**:

1. **Auto-populate from git** (if not provided):
   ```bash
   git diff --name-status HEAD~1        # Files touched
   git diff HEAD~1 -- src/ | head -100  # Sample of changes
   git log -1 --format=%B              # Commit message
   ```

2. **Fill snapshot structure**:
   ```yaml
   context_snapshot:
     implementation: "[One-paragraph description of the change or feature]"
     primary_goal: "[What success was supposed to look like]"
     files_touched:
       created: []
       modified: []
       deleted: []
     dependencies_added: []
     risk_mode: "[sandbox/elevated/production]"
     complexity: "[low/medium/high]"
   ```

3. **Document**:
   - What was I trying to accomplish?
   - What felt tricky or uncertain during implementation?
   - What shortcuts did I take?
   - Where did I improvise vs follow patterns?

**Output**:
- Context snapshot filled
- Risk mode identified
- Complexity assessed
- Ready for detection phase

---

## PLAY: Parallel Detection Sweep

**Purpose**: Fan out 8-12 parallel detection agents to find issues across three phases.

**CRITICAL**: Spawn ALL detection agents in ONE message. Do NOT run sequentially.

### Detection Categories

**Phase 1: Correctness Bugs**
- TODO/FIXME/HACK/XXX markers
- `: any` type annotations
- Magic numbers (hardcoded 2+ digit numbers)
- Empty catch blocks
- `.then()` without `.catch()`

**Phase 2: Security Vulnerabilities**
- SQL injection patterns (query with template literals)
- `exec`/`spawn`/`execSync` usage
- `innerHTML` or `dangerouslySetInnerHTML`
- Hardcoded passwords/secrets/api_keys/tokens
- Path traversal in file operations

**Phase 3: Structural Issues**
- Functions >50 lines
- Cyclomatic complexity >10
- Deep nesting >4 levels
- Code duplication >3 occurrences
- `readFileSync`/`writeFileSync` (sync I/O)
- N+1 patterns (loops with individual queries)
- Phase references in production code (Phase X.Y markers)
- Untested exported functions

### Parallel Execution Pattern

**SPAWN ALL AT ONCE - Do not wait between calls:**

```
Task(model: "haiku", prompt: "PHASE 1.1 Correctness: Find TODO/FIXME/XXX/HACK markers in src/**/*.ts. Return file:line, match")
Task(model: "haiku", prompt: "PHASE 1.1 Correctness: Find ': any' type annotations in src/**/*.ts. Return file:line")
Task(model: "haiku", prompt: "PHASE 1.1 Correctness: Find magic numbers (hardcoded 2+ digit numbers) in src/**/*.ts excluding tests")
Task(model: "haiku", prompt: "PHASE 1.2 Error Handling: Find empty catch blocks in src/**/*.ts. Return file:line")
Task(model: "haiku", prompt: "PHASE 1.2 Error Handling: Find .then() without .catch() in src/**/*.ts. Return file:line")
Task(model: "haiku", prompt: "PHASE 2.1 Security: Find SQL injection patterns (query with template literals) in src/**/*.ts")
Task(model: "haiku", prompt: "PHASE 2.1 Security: Find exec/spawn/execSync usage in src/**/*.ts")
Task(model: "haiku", prompt: "PHASE 2.1 Security: Find innerHTML or dangerouslySetInnerHTML in src/**/*.ts")
Task(model: "haiku", prompt: "PHASE 2.3 Secrets: Find hardcoded password/secret/api_key/token in src/**/*.ts excluding test/mock")
Task(model: "haiku", prompt: "PHASE 3.1 Maintainability: Find functions >50 lines in src/**/*.ts. Return function name, file:line, count")
Task(model: "haiku", prompt: "PHASE 3.2 Performance: Find readFileSync/writeFileSync usage in src/**/*.ts")
Task(model: "haiku", prompt: "PHASE 3.2 Performance: Find N+1 patterns (loops with individual queries/fetches) in src/**/*.ts")
```

**Minimum 8-12 parallel agents per harvest. All run concurrently in ~5 seconds.**

**Output**: Aggregated findings from all detection agents

---

## PLAY: Analysis & Triage

**Purpose**: Classify all findings by severity and complexity. Prepare for user decision.

**Input**: Raw detection results from all 12 agents

**Steps**:

1. **Classify by severity**:
   - 🔴 **CRITICAL**: Security vulnerability, data loss, system crash
   - 🟡 **HIGH**: Incorrect behavior, significant user impact
   - 🟢 **MEDIUM**: Degraded experience, workaround exists
   - ⚪ **LOW**: Minor issue, cosmetic, optimization

2. **Assess complexity**:
   - **Low**: Single file, obvious fix, isolated change
   - **Medium**: Multiple files, some reasoning, testable
   - **High**: Architectural, cross-cutting, needs planning

3. **Determine risk**:
   - **Low**: Isolated, easy rollback, non-critical
   - **Medium**: Moderate blast radius, affects some users
   - **High**: Wide blast radius, data integrity, security

4. **Triage recommendation**:
   - **Low complexity + any severity** → 🔧 Patch Now
   - **Medium complexity + HIGH severity** → 🔧 Patch Now
   - **Medium complexity + MEDIUM/LOW** → 📋 Backlog
   - **High complexity + HIGH severity** → 📦 Build Phase (urgent)
   - **High complexity + MEDIUM/LOW** → 📦 Build Phase (scheduled)

5. **Create finding cards** (see output format below)

**Output**: Findings sorted by criticality, each with:
- Title and file location
- Severity, Complexity, Risk assessment
- What's wrong (1-2 sentences)
- Evidence/example
- Why it matters
- Specific recommendation (with code snippet if applicable)

---

## PLAY: Context-Aware Reflection

**Purpose**: Answer implementation-specific questions based on what you just built.

**Steps**:

1. **Edge Cases**:
   - What edge cases did I handle?
   - Which did I skip?
   - What happens at boundaries (empty input, max size, null)?

2. **Code Quality**:
   - Where did I copy-paste code? Is there duplication?
   - What error scenarios are NOT handled?
   - Did I add logging at failure points?

3. **Dependencies**:
   - What happens if [key dependency] is unavailable?
   - Are all external calls wrapped in try/catch?
   - Do I validate all external inputs?

4. **Technology-Specific** (MCP Servers):
   - Are all tool inputs validated with Zod schemas?
   - Do all tools return `{ content: [...] }` format?
   - Are error responses setting `isError: true`?
   - Are tool descriptions clear for LLM consumption?

5. **Technology-Specific** (APIs):
   - Are all endpoints documented?
   - Are error responses consistent?
   - Is rate limiting in place?

6. **Technology-Specific** (CLI tools):
   - Is help text complete?
   - Are all flags documented?
   - Do error messages guide the user?

**Output**: Answers to all applicable questions (none are skippable)

---

## PLAY: Feature Opportunities

**Purpose**: Capture ideas that emerged from the work, not planned upfront.

**Categories**:

| Category | Examples |
|----------|----------|
| Extensions | Obvious additions that compound value |
| Automation | Opportunities revealed by repetition |
| Metrics | New insights now possible |
| Efficiency | Features that reduce future manual effort |

**Important**: Do not design these fully. Capture them as:
- **Title**: One clear noun phrase
- **Idea**: 1-2 sentences
- **Why It Matters**: How this adds value or improves efficiency
- **Complexity**: Low/Medium/High (rough estimate)

**Output**: 1-3 feature seeds

---

## PLAY: Meta-Assessment

**Purpose**: Reflect on implementation quality from a bird's-eye view.

**Answer these four questions**:

1. **Most Fragile**: What part of this implementation feels most fragile? What would break first under load or edge cases?

2. **Surprisingly Solid**: What part is unexpectedly robust? What feels battle-tested?

3. **Future Confusion Point**: If this were revisited in 6 months, what would confuse a new reader first?

4. **Hidden Assumption**: What assumption did this work quietly depend on? (e.g., "assumes all inputs are validated", "assumes database is always available")

**Output**: One-sentence answers to all four questions

---

## PLAY: Present Findings & Ask for Direction

**Purpose**: Show all findings to user and get explicit approval before fixing anything.

**CRITICAL PROTOCOL**:

1. **DO NOT auto-fix** - Assessment only at this stage
2. **PRESENT all findings** in full detail (not a summary)
3. **Ask for direction** with clear options
4. **WAIT** for user approval before proceeding

**Structure**:

```
HARVEST REPORT
==============

Implementation: [Feature/System Name]
Date: [YYYY-MM-DD]
Harvester: Claude Code
Risk Mode: [sandbox/elevated/production]

---

EXECUTIVE SUMMARY

[2-3 sentences on implementation health, key risks, recommended action]

Overall Risk Level: [LOW/MEDIUM/HIGH]
Total Findings: [X] critical, [Y] high, [Z] medium, [W] quick wins, [V] feature seeds

---

## FINDINGS (Sorted by Criticality)

### 🔴 CRITICAL ([n] findings)

**C1: [Title]**

File: [path:line]
Severity: 🔴 CRITICAL
Complexity: [Low/Medium/High]
Risk: [Data loss / Security / System crash / etc]
Action: [TBD - waiting for your decision]

**What's Wrong:**
[Concise description. 1-2 sentences.]

**Example/Evidence:**
[Code snippet or concrete evidence]

**Why This Matters:**
[Impact if not fixed]

**Specific Recommendation:**
[Clear, actionable fix]

---

### 🟡 HIGH ([n] findings)
[Repeat card format]

### 🟢 MEDIUM ([n] findings)
[Repeat card format]

### ⚪ LOW ([n] findings)
[Repeat card format]

### ⚡ QUICK WINS ([n] items)

**Q1: [Title]**

File: [path:line]
Effort: 2-5 min
Impact: [Low/Medium/High]
Action: [TBD]

**Opportunity:**
[What can be improved quickly]

---

### 💡 FEATURE SEEDS ([n] items)

**F1: [Title]**

Value: [Low/Medium/High]
Complexity: [Low/Medium/High]

**Idea:**
[What could be built]

**Why It Matters:**
[How this adds value]

---

## META-ASSESSMENT

- **Most Fragile**: [Answer]
- **Surprisingly Solid**: [Answer]
- **Future Confusion Point**: [Answer]
- **Hidden Assumption**: [Answer]

---

## TRIAGE PROPOSAL (AWAITING YOUR APPROVAL)

Overall Risk: [LOW/MEDIUM/HIGH]

🔧 PROPOSED Patch Now: [n] items ([X] min)
   - C1: [title] (Low complexity)
   - H1: [title] (Low complexity)
   - Q1: [title] (2 min)

📋 PROPOSED Backlog: [n] items
   - M1: [title] (Medium complexity)
   - L1: [title] (Cosmetic)

📦 PROPOSED Build Phase: [n] items
   - Phase-X: [title] (High complexity, blocks downstream)

---

AWAITING YOUR DECISION:

What would you like to do?

1. Fix all Patch Now items ([n] items, [X] min total)
2. Fix critical only ([n] items)
3. Fix critical + high ([n] items)
4. Present detailed analysis (already shown above)
5. Something else

→ Not proceeding with any fixes until you approve
```

**Output**: Complete harvest report + awaiting user direction

---

## PLAY: Execute Approved Fixes

**Purpose**: Only execute fixes the user explicitly approved.

**CRITICAL**: Do NOT execute until user responds to the triage proposal.

**Steps**:

1. **Wait for user response** to the triage proposal

2. **Parse approval**:
   - If "Fix all Patch Now" → Execute all 🔧 items
   - If "Fix critical only" → Execute only 🔴 CRITICAL items marked 🔧
   - If "Fix critical + high" → Execute 🔴 + 🟡 items marked 🔧
   - If something else → Follow user's custom direction

3. **For each approved item**:

   **If 🔧 Patch Now (simple fix)**:
   - Fix immediately via direct edit or brief Codex spec
   - Run validation (tests, typecheck)
   - Report result

   **If 📋 Backlog**:
   - Document in `_build/prompts/BACKLOG.md`:
   ```markdown
   ## [ID] [Title]
   - **Source**: /harvest [date]
   - **Severity**: [CRITICAL/HIGH/MEDIUM/LOW]
   - **Complexity**: [Low/Medium/High]
   - **Risk**: [What breaks if we don't fix this]
   - **Description**: [What needs to be done]
   - **File(s)**: [path:line]
   ```
   - Do NOT fix now
   - Mark as 📋 in report

   **If 📦 Build Phase**:
   - Create `_build/prompts/Phase-#-[title].md` with full spec
   - Do NOT implement now
   - Mark as 📦 in report
   - Later: Run `/bryan` to validate spec
   - Later: Run `/-codex_prompt` to generate execution spec

4. **Validation** (if fixes applied):
   ```bash
   npm test
   npm run typecheck
   npm run lint
   ```

5. **Report final status**:
   - ✅ Fixes applied
   - ✅ Tests passing
   - 📋 Backlog items documented
   - 📦 Build Phase specs created
   - 🔗 Next steps

**Output**:
- All fixes executed (if approved)
- Validation complete
- Backlog/Phase items documented
- User ready for next step

---

## DECISION MATRIX: Patch Now vs Backlog vs Build Phase

| Complexity | High Severity | Medium Severity | Low Severity |
|------------|---------------|-----------------|--------------|
| **Low** | 🔧 Patch Now | 🔧 Patch Now | 📋 Backlog |
| **Medium** | 🔧 Patch Now | 📋 Backlog | 📋 Backlog |
| **High** | 📦 Build Phase (urgent) | 📦 Build Phase (scheduled) | 📋 Backlog |

---

## INTEGRATION WITH WORKFLOW

**When to trigger /harvest**:

1. After completing any feature >100 lines changed
2. Before merging to main branch
3. After refactoring existing code
4. Before production deployment
5. After completing a Phase specification

**Workflow**:

```
Implementation → /harvest → Findings → User Decision → Execute → Continue
```

**For Build Phase items downstream**:

```
/harvest (creates Phase spec)
  → /bryan (validates spec)
  → /-codex_prompt (execution spec)
  → Codex (implements)
```

---

## EXAMPLE HARVEST REPORT

See `/harvest.md` command file for example output showing card-based format in action.

---

## PREREQUISITES

Project should have:

```
project-root/
├── _build/
│   └── prompts/
│       ├── BACKLOG.md           # Backlog items go here
│       └── Phase-#-[title].md   # Build Phase specs
├── src/                         # Source code to analyze
└── tests/                       # Test files
```

**Create if missing**:
```bash
mkdir -p _build/prompts
touch _build/prompts/BACKLOG.md
```

---

## HARVEST CHECKLIST

Before declaring harvest complete:

- [ ] **Context Snapshot**: Filled with implementation details
- [ ] **All Detection Phases**: Executed (Phases 1-3 in parallel)
- [ ] **All Findings Presented**: Each with Severity, Complexity, Risk
- [ ] **User Asked for Direction**: Triage proposal presented
- [ ] **AWAITING USER APPROVAL**: Not proceeding until user responds
- [ ] **All Findings Triaged**: Each has Action (🔧 / 📋 / 📦) per user guidance
- [ ] **Fixes Executed**: Only items user approved
- [ ] **Validation**: Tests/typecheck passed (if fixes applied)
- [ ] **Backlog Items**: Added to BACKLOG.md (if user approved)
- [ ] **Build Phase Specs**: Created if needed (if user approved)

---

## PHILOSOPHY

This playbook converts momentum into clarity, not waste.

- The moment after heavy work is when context is richest
- Blind spots are easiest to catch while details are fresh
- Capturing issues now prevents expensive rediscovery later
- Brutal honesty now saves painful debugging later
- **User approval gate prevents auto-fixing** (respects user agency)

**Avoid speculation. Anchor everything in what was built.**

---

## REFERENCES

- **Harvest Command**: [../harvest.md](../harvest.md)
- **Backlog**: [../../../_build/prompts/BACKLOG.md](../../../_build/prompts/BACKLOG.md)
- **OWASP Top 10**: https://owasp.org/Top10/
- **Node.js Security**: https://cheatsheetseries.owasp.org/cheatsheets/Nodejs_Security_Cheat_Sheet.html
