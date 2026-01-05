# Lean Skill Playbook Protocol

> **Purpose**: Template for creating lean Claude Code skills with playbook-backed logic
> **Version**: 1.3.0
> **Last Updated**: 2025-12-26

---

## Overview

This protocol defines how to structure Claude Code skills for minimal context load while maintaining full capability through playbook delegation.

**Architecture**: Lean skill stub → Playbook (detail) → Execution

---

## File Layout

```
.claude/
├── commands/
│   └── <skill-name>.md             # Lean stub (~50-80 lines)
└── commands/playbooks/
    ├── .claudeignore               # Contains: *
    └── <skill-name>-playbook.md    # Full reference (200-500+ lines)
```

### .claudeignore

The playbooks folder MUST contain a `.claudeignore` with `*` to prevent automatic context injection:

```
*
```

This ensures:
- Playbooks are NOT injected into global context
- They are pulled ONLY when invoked
- Context stays clean and cheap

---

## Skill Stub Template

### `.claude/commands/<skill-name>.md`

```markdown
---
description: <One-line description of what this skill does>
tags: [tag1, tag2, tag3]
---

<Brief description - 1-2 sentences>

## Usage

\`\`\`bash
/<skill-name> <required-arg>                    # Basic usage
/<skill-name> <arg> --flag                      # With optional flag
/<skill-name> path/to/files/*.md                # Batch usage
\`\`\`

## Quick Reference

<Essential reference table - tiers, scores, key metrics>

| Key | Value |
|-----|-------|
| ... | ... |

## Core Logic Summary

<Bullet points of what happens - 5-10 lines max>

- Step 1
- Step 2
- Step 3

## Flags

| Flag | Description |
|------|-------------|
| `--flag-name` | What it does |

## Playbook

Full protocol: `.claude/commands/playbooks/<skill-name>-playbook.md`
```

**Target size**: 50-80 lines

---

## Playbook Template

### `.claude/commands/playbooks/<skill-name>-playbook.md`

```markdown
# <Skill Name> Playbook - Complete Reference

> **Purpose**: Full execution protocol for <skill>
> **Version**: X.Y.Z
> **Last Updated**: YYYY-MM-DD

---

## INPUTS

- `target` (required): Description
- `--flag` (optional): Description

---

## OUTPUTS

Structured report containing:
- Result 1
- Result 2
- Actionable recommendations

---

## EXECUTION MODEL

Sequential/parallel pass description.

\`\`\`
Pass 1 → Pass 2 → (optional) Pass 3
\`\`\`

---

## PASS 1: <NAME>

### Purpose
What this pass accomplishes.

### Checks
- [ ] Check 1
- [ ] Check 2

### Output
- PASS / FAIL
- Details

### Failure Rule
What happens if this pass fails.

---

## PASS 2: <NAME>

### Purpose
What this pass accomplishes.

### Scoring/Evaluation
| Dimension | Weight | Target |
|-----------|--------|--------|
| ... | ... | ... |

### Output
- Score
- Classification

---

## PASS N: <NAME> (OPTIONAL)

**Enabled only when `--flag` is present**

### Purpose
Extended validation/checks.

### Output
- Extended results

---

## OUTPUT FORMAT

\`\`\`
<SKILL> REPORT
==============

File: <path>
Date: <date>

PASS 1: <NAME> .......... PASS/FAIL
PASS 2: <NAME> .......... SCORE (TIER)

DEDUCTIONS
- Severity (-N): Reason

FINAL SCORE: X / Y
FINAL TIER: <icon> <tier>

RECOMMENDATIONS
- Action item 1
- Action item 2
\`\`\`

---

## DOES ✅

- Thing 1
- Thing 2
- Thing 3

## DOES NOT ❌

- Thing 1
- Thing 2
- Thing 3

---

## STOP CONDITION

Return report only.
Do not propose edits unless explicitly asked.

---

## REFERENCES

- [Reference 1](path/to/file.md)
- [Reference 2](path/to/file.md)
```

**Target size**: 200-500 lines (no upper limit for complex skills)

---

## Model Delegation

Assign the right model to each pass based on task complexity:

| Task Type | Model | Rationale |
|-----------|-------|-----------|
| Syntax checks, placeholder detection | Haiku | Fast, cheap validation |
| Quality scoring, reasoning | Sonnet | Balanced cost/capability |
| External verification, web fetches | Sonnet | Needs reasoning + tools |
| Architecture decisions | Opus | Strategic (rare) |
| Pure code generation | Codex | $0 cost from specs |

**Default pattern**: Haiku (validation) → Sonnet (scoring) → Optional Sonnet (verification)

---

## Design Principles

### 1. Separation of Concerns

| Layer | Purpose | Size |
|-------|---------|------|
| Skill stub | Quick reference, usage, invocation | 50-80 lines |
| Playbook | Full protocol, checklists, output formats | 200-500+ lines |

### 2. Context Economics

- **Skill stubs**: Always loaded into context (~500-800 tokens)
- **Playbooks**: Loaded on-demand only (~2000-5000 tokens)
- **Savings**: 60-90% context reduction for unused skills

### 3. Deterministic Execution

Playbooks define:
- Explicit inputs/outputs
- Sequential passes with clear dependencies
- Failure rules and stop conditions
- Output contracts (machine + human readable)

### 4. Capability Boundaries

Every playbook MUST define:
- **DOES**: Explicit capabilities
- **DOES NOT**: Explicit limitations
- **STOP CONDITION**: When to halt and return

---

## Conversion Checklist

When converting an existing skill to lean + playbook:

1. [ ] Create playbooks directory if not exists
2. [ ] Add `.claudeignore` with `*`
3. [ ] Extract detailed content to `<skill>-playbook.md`
4. [ ] Reduce skill stub to essential reference only
5. [ ] Add playbook path reference in skill stub
6. [ ] Verify skill stub is under 80 lines
7. [ ] Test skill invocation still works

---

## Example: Before/After

### Before (683 lines in skill)

```
.claude/commands/kbgen-validate.md  # 683 lines - everything inline
```

### After (60 + 289 lines, separated)

```
.claude/commands/kbgen-validate.md                      # 60 lines - lean stub
.claude/commands/playbooks/kbgen-validate-playbook.md   # 289 lines - full ref
.claude/commands/playbooks/.claudeignore                # * - context excluded
```

**Context savings**: ~90% when skill not invoked

---

## Naming Conventions

| File Type | Pattern | Example |
|-----------|---------|---------|
| Skill stub | `<name>.md` | `kbgen-validate.md` |
| Playbook | `<name>-playbook.md` | `kbgen-validate-playbook.md` |
| Extended playbook | `<name>-<pass>-playbook.md` | `kbgen-validate-pass3-playbook.md` |

---

## When to Use Playbooks

**Use playbook separation when**:
- Skill logic exceeds 100 lines
- Multiple passes or phases
- Complex scoring/evaluation
- Extensive checklists or tables
- Detailed output formats

**Keep inline when**:
- Simple single-action skills
- Under 80 lines total
- No complex logic or phases

---

## Playbook Invocation

When Claude executes a skill with a playbook reference:

1. Read the lean skill stub (always in context)
2. If detailed execution needed, read the playbook
3. Execute according to playbook protocol
4. Return structured output per playbook contract

The playbook is NOT auto-loaded - it's read on-demand during execution.

---

## Quality Checklist

### Skill Stub Quality
- [ ] Under 80 lines
- [ ] Has usage examples
- [ ] Has quick reference table
- [ ] Links to playbook
- [ ] Frontmatter has description and tags

### Playbook Quality
- [ ] Has version and date
- [ ] Defines INPUTS and OUTPUTS
- [ ] Defines execution model
- [ ] Each pass has purpose, checks, output, failure rule
- [ ] Has DOES/DOES NOT section
- [ ] Has STOP CONDITION
- [ ] Has output format template
- [ ] Model assigned to each pass

---

## Self-Test Protocol

Before marking a skill/playbook complete:

1. **Invoke the skill** with a real target
2. **Verify output** matches the OUTPUT FORMAT template
3. **Test edge cases**: empty input, invalid path, missing flags
4. **Confirm playbook loads** when referenced from stub

```bash
# Example validation
/<skill-name> path/to/test-file.md
# Should produce structured output, not errors
```

**Pass criteria**: Skill executes, output matches contract, no unhandled errors.

---

## This Playbook: DOES / DOES NOT

### What This Protocol DOES ✅

- Defines file structure for lean skills + playbooks
- Provides templates for both layers
- Sets size targets and naming conventions
- Establishes quality checklists
- Documents model delegation patterns

### What This Protocol DOES NOT ❌

- Generate skills automatically
- Enforce rules at runtime
- Replace domain-specific playbook content
- Validate playbook correctness (use /bryan for that)

---

## Versioning

Follow semantic versioning for playbooks:

- **PATCH** (1.0.X): Typo fixes, clarifications, no behavior change
- **MINOR** (1.X.0): New sections, expanded checklists, backward compatible
- **MAJOR** (X.0.0): Breaking changes to output format, removed sections, incompatible

---

## /bryan Integration

Before finalizing any skill/playbook:

1. Run `/bryan audit <playbook-path>` to check completeness
2. Address any BLOCKER or HIGH gaps
3. POLISH gaps are optional but recommended
4. Re-audit after significant changes

---

## STOP CONDITION

Return the generated skill stub and playbook templates.

Do NOT:
- Auto-create files without user confirmation
- Modify existing skills without explicit request
- Execute the generated skill (only generate templates)
