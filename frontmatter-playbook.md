---
type: playbook
purpose: Standardize metadata frontmatter across all document types for machine-parseability and human readability
version: 1.0
status: authoritative
last_updated: 2026-02-08
tags: [frontmatter, metadata, playbook, standards, documentation]
---

# Frontmatter Playbook

> **Purpose**: Standardize metadata frontmatter across all document types for machine-parseability and human readability
> **Version**: 1.0
> **Status**: Authoritative
> **Last Updated**: 2026-02-08

---

## Mental Model

Frontmatter is **structured metadata** that makes documents:
- **Machine-parseable** (search, filter, index)
- **Human-readable** (quick context at a glance)
- **Version-controlled** (track changes over time)
- **Discoverable** (faceted search, classification)

```
Document Type → Frontmatter Pattern → Fields → Format → Tools
       ↓               ↓                 ↓        ↓        ↓
   "What is it"    "How to tag"    "Metadata"  "YAML"  "Search"
```

**The Frontmatter Hierarchy**:
```
Essential (MUST) → type, version, status
Standard (SHOULD) → purpose, last_updated, tags
Rich (MAY) → custom fields per document type
```

---

## Frontmatter by Document Type

### Type 1: Agent Profiles (Behavior Contracts)

**Pattern**: HTML comments at top of file

**Format**:
```markdown
<!--
AGENT_READ_FIRST (MUST)
- Read this entire document top-to-bottom before responding.
- Treat this file as the authoritative behavior contract.
-->

<!--
<AGENT_NAME>_ENTRYPOINT
Agent: <agent-identifier>
Version: v<major>.<minor>
Mode: <primary-mode>
Behavior: <behavior-pattern>
Start State: <initial-state>
Authority: <authority-level>
Status: <status>
Last Updated: <YYYY-MM-DD>
-->

# <Agent Name> — <Subtitle>
```

**Required Fields**:
- `Agent:` - Identifier (melissa, hal9000, rosie-x)
- `Version:` - Semantic version (v1.2, v3.0)
- `Mode:` - Primary operating mode
- `Behavior:` - Behavioral pattern
- `Start State:` - Initial state on invocation
- `Authority:` - Authority level (Advisory, Execution, Code-Only, Full)

**Optional Fields**:
- `Status:` - Document maturity
- `Last Updated:` - Date of last modification
- `Replaces:` - Previous version reference
- `Requires:` - Dependencies

**Example**: melissa-ai-spec-whisperer.md, hal9000.md, rosie-x.md

---

### Type 2: Playbooks (Process Definitions)

**Pattern**: YAML frontmatter + blockquote

**Format**:
```markdown
---
type: playbook
purpose: <one-line purpose statement>
version: <major>.<minor>
status: <draft|review|authoritative|deprecated>
last_updated: <YYYY-MM-DD>
tags: [<tag1>, <tag2>, <tag3>]
---

# <Playbook Title>

> **Purpose**: <one-line purpose statement>
> **Version**: <major>.<minor>
> **Status**: <status>
> **Last Updated**: <YYYY-MM-DD>

---

## Mental Model
```

**Required Fields**:
- `type:` - Always "playbook"
- `purpose:` - Clear statement of intent
- `version:` - Semantic version number
- `status:` - Document maturity
- `last_updated:` - ISO date (YYYY-MM-DD)

**Optional Fields**:
- `tags:` - Keywords array
- `id:` - Unique identifier (e.g., SCP-001)
- `applies_to:` - Scope or project
- `audience:` - Target users
- `replaces:` - Previous version

**Example**: prd-playbook.md, frd-playbook.md, github-project-setup.md

---

### Type 3: Standards & Profiles (Reference Documents)

**Pattern**: Header comments

**Format**:
```markdown
# <Document Title>
# Version: v<major>.<minor>
# Status: <Locked|Draft|Review>
# Scope: <applicability>
# Last Updated: <YYYY-MM-DD>

---

## <First Section>
```

**Required Fields**:
- `Version:` - Semantic version with 'v' prefix
- `Status:` - Document state
- `Scope:` - What this document covers

**Optional Fields**:
- `Last Updated:` - Date
- `Authority:` - Who can modify
- `Replaces:` - Previous version

**Example**: bryan-developer-profile.md

---

### Type 4: Templates (Reusable Structures)

**Pattern**: YAML frontmatter + usage instructions

**Format**:
```markdown
---
type: template
use_for: <intended-use>
version: <major>.<minor>
last_updated: <YYYY-MM-DD>
tags: [template, <category>]
---

# <Template Name>

> **Type**: Template
> **Use For**: <intended-use>
> **Version**: <major>.<minor>

**Usage Instructions**:
1. Copy this template
2. Replace all `<placeholders>`
3. Delete usage instructions

---

[Template content begins]
```

**Required Fields**:
- `type:` - Always "template"
- `use_for:` - Clear description of use case
- `version:` - Version number

**Optional Fields**:
- `last_updated:` - ISO date
- `based_on:` - Source standard
- `examples:` - Reference implementations
- `tags:` - Keywords

**Example**: adr-template.md

---

### Type 5: Guides & Reference (Documentation)

**Pattern**: YAML frontmatter + blockquote

**Format**:
```markdown
---
type: guide
purpose: <what this guide covers>
version: <major>.<minor>
last_updated: <YYYY-MM-DD>
tags: [guide, <category>]
---

# <Guide Title>

> **Purpose**: <what this guide covers>
> **Version**: <major>.<minor>
> **Last Updated**: <YYYY-MM-DD>

---
```

**Required Fields**:
- `type:` - Always "guide"
- `purpose:` - What the guide covers
- `version:` - Version number

**Optional Fields**:
- `last_updated:` - Date
- `prerequisites:` - Required knowledge
- `related:` - Links to related guides
- `tags:` - Keywords

**Example**: mermaid-guide.md

---

### Type 6: Specifications (PRD, FRD, TechSpec)

**Pattern**: YAML frontmatter + blockquote

**PRD Format**:
```markdown
---
type: prd
product: <product-name>
version: <major>.<minor>
status: <draft|review|approved|locked>
last_updated: <YYYY-MM-DD>
success_metrics: [<metric1>, <metric2>]
tags: [prd, requirements, <domain>]
related_docs:
  frd: <frd-filename>
  techspec: <techspec-filename>
---

# PRD: <Product Name>

> **Product**: <product-name>
> **Version**: <major>.<minor>
> **Status**: <status>
> **Last Updated**: <YYYY-MM-DD>

---
```

**FRD Format**:
```markdown
---
type: frd
feature: <feature-name>
version: <major>.<minor>
status: <draft|review|approved|locked>
last_updated: <YYYY-MM-DD>
prd_reference: <prd-filename>
tags: [frd, specifications, <domain>]
---

# FRD: <Feature Name>

> **Feature**: <feature-name>
> **Version**: <major>.<minor>
> **Status**: <status>
> **PRD Reference**: [<prd-name>](<prd-link>)

---
```

**Example**: See prd-playbook.md, frd-playbook.md

---

## Standard Fields (All Types)

| Field | Format | Required | Description |
|-------|--------|----------|-------------|
| `type` | string | Yes | Document type (playbook, guide, template, etc.) |
| `version` | `<major>.<minor>` | Yes | Semantic version number |
| `purpose` | single line | Recommended | Clear statement of intent |
| `status` | enum | Recommended | Document maturity level |
| `last_updated` | `YYYY-MM-DD` | Recommended | Date of last modification |
| `tags` | array | Recommended | Keywords for search/classification |

---

## Status Values (Standardized)

| Status | Meaning | When to Use |
|--------|---------|-------------|
| `draft` | Work in progress, not authoritative | Initial creation, significant rewrites |
| `review` | Ready for review, pending approval | Submitted for feedback |
| `authoritative` | Canonical, production-ready | Approved and in active use |
| `locked` | Frozen, requires explicit unlock | Finalized contracts, approved specs |
| `deprecated` | Obsolete, replaced by newer version | Superseded by new version |
| `archived` | Historical reference only | No longer applicable but kept for history |

---

## Authority Levels (Agent Profiles)

| Level | Meaning | Capabilities |
|-------|---------|--------------|
| `advisory` | Can suggest, not execute | Question-asking, analysis |
| `execution` | Can execute non-code tasks | File operations, orchestration |
| `code-only` | Sole authority for code changes | Write/edit code exclusively |
| `full` | Complete authority in domain | All operations in scope |

---

## Version Numbering Standard

Follow semantic versioning:

- **Major version** (`v1.0` → `v2.0`): Breaking changes, incompatible updates
- **Minor version** (`v1.0` → `v1.1`): Additive changes, new features, compatible updates
- **Patch version** (`v1.1.0` → `v1.1.1`): Bug fixes, typos, clarifications (optional)

**Examples**:
- `v1.0` - Initial authoritative release
- `v1.1` - Added new section, backward compatible
- `v2.0` - Changed authority model, breaking change
- `v1.2.1` - Fixed typo in example (patch optional)

---

## Application Procedure (Autonomous Execution)

This procedure can be executed autonomously by AI agents to apply frontmatter standards to existing files.

### Step-by-Step Application

**Input**: List of files to update (e.g., `prd-playbook.md`, `frd-playbook.md`)

**Process**:

1. **Read each file completely** before editing (MANDATORY)
   - Use Read tool to load full file contents
   - Never attempt Edit without prior Read in same session

2. **Identify document type** from patterns:
   - Playbook: Contains "playbook" in filename or purpose
   - Agent Profile: Contains agent behavior contract
   - Standard/Profile: Contains version pinning or reference material
   - Template: Contains reusable structure
   - Guide: Contains "guide" in filename or purpose

3. **Extract existing metadata** from blockquote or header:
   - Purpose statement (line with `> **Purpose**:` or `**Purpose**:`)
   - Version number (line with `**Version**:` or `Version:`)
   - Status (line with `**Status**:` or `Status:`)
   - Last Updated date (line with `**Last Updated**:`)
   - Tags (infer from content: keywords, domain terms)

4. **Generate YAML frontmatter block**:
   ```yaml
   ---
   type: <document-type>
   purpose: <extracted-purpose-statement>
   version: <extracted-version>
   status: <lowercase-status>
   last_updated: <YYYY-MM-DD-current-date>
   tags: [<tag1>, <tag2>, <tag3>]
   ---
   ```

5. **Construct replacement text**:
   - YAML block (from step 4)
   - Blank line
   - Original title line (e.g., `# PRD Playbook`)
   - Blank line
   - Original blockquote section (preserve for human readability)
   - Blank line
   - Horizontal rule `---`
   - Rest of document unchanged

6. **Apply edit** using Edit tool:
   - `old_string`: From start of file to end of first `---` (original content)
   - `new_string`: Constructed replacement with YAML + blockquote
   - `replace_all`: false

7. **Verify changes**:
   - YAML frontmatter present at top
   - Blockquote preserved below
   - No content lost
   - Proper formatting (blank lines, horizontal rules)

8. **Repeat** for each file in input list

9. **Commit all changes together**:
   ```bash
   git add <file1> <file2> <file3>
   git commit -m "docs: standardize frontmatter for N playbooks with YAML format

   - Added YAML frontmatter blocks with --- delimiters
   - Updated last_updated dates to YYYY-MM-DD
   - Added structured metadata (type, purpose, version, status, tags)
   - Kept blockquote format below YAML for human readability

   Files updated:
   - <file1>
   - <file2>
   - <file3>

   Follows frontmatter-playbook.md"
   ```

### Example Transformation

**Before**:
```markdown
# PRD (Product Requirements Document) Playbook

> **Purpose**: Guide the creation of PRDs
> **Version**: 1.0
> **Status**: Authoritative
> **Last Updated**: 2024-12-31

---

## Mental Model
```

**After**:
```markdown
---
type: playbook
purpose: Guide the creation of PRDs that clearly define the problem, users, and success criteria
version: 1.0
status: authoritative
last_updated: 2026-02-08
tags: [prd, playbook, requirements, specifications]
---

# PRD (Product Requirements Document) Playbook

> **Purpose**: Guide the creation of PRDs that clearly define the problem, users, and success criteria
> **Version**: 1.0
> **Status**: Authoritative
> **Last Updated**: 2026-02-08

---

## Mental Model
```

### Error Handling

**Error**: File not read before editing
- **Fix**: Read file first with Read tool, then retry Edit

**Error**: Cannot find existing metadata
- **Fix**: Manual review required, ask user for missing fields

**Error**: Ambiguous document type
- **Fix**: Default to `playbook` if filename contains "playbook", else ask user

---

## Best Practices

### DO ✅

1. **Use consistent formatting** within document type
2. **Update version** on every significant change
3. **Set status** to reflect current maturity
4. **Write clear purpose** statements (one line, outcome-focused)
5. **Use ISO dates** (`YYYY-MM-DD`) for timestamps
6. **Reference dependencies** explicitly
7. **Mark deprecated** documents with replacement links
8. **Use semantic versioning** consistently

### DON'T ❌

1. **Don't mix frontmatter styles** in same document
2. **Don't use vague status** values (use standardized enum)
3. **Don't leave version** at 0.1 forever
4. **Don't skip purpose** statements (readers need context)
5. **Don't use relative dates** ("last month", "recently")
6. **Don't create new status** values without updating this playbook
7. **Don't forget to update** "Last Updated" on changes
8. **Don't use non-semantic** version numbers (dates, hashes)

---

## Migration Strategy

### For Existing Documents

When updating legacy documents to this standard:

1. **Identify document type** from list above
2. **Add or update frontmatter** to match pattern
3. **Preserve existing version** unless making breaking changes
4. **Set appropriate status** based on current usage
5. **Add `last_updated`** field with current date
6. **Commit with clear message**: `docs: standardize frontmatter for <filename>`

### For New Documents

When creating new documents:

1. **Choose document type** category
2. **Copy frontmatter pattern** from this playbook
3. **Start at version** `1.0` (or `v1.0` for agents)
4. **Set status** to `draft` initially
5. **Include purpose** statement
6. **Add to repository index** (README.md)

---

## Validation Checklist

Use this checklist when adding/updating frontmatter:

- [ ] Frontmatter matches document type pattern
- [ ] Version number present and follows semantic versioning
- [ ] Status value is from standardized enum
- [ ] Purpose statement is clear and concise
- [ ] Last Updated date is current (if modified)
- [ ] No custom fields without documentation
- [ ] No typos in field names
- [ ] Proper markdown formatting (consistent with document type)

### Post-Application Validation

After applying to files:

1. **Verify YAML syntax**:
   ```bash
   for file in *.md; do
     head -n 20 "$file" | grep -q "^---$" && echo "$file: ✅" || echo "$file: ❌"
   done
   ```

2. **Check date format** (YYYY-MM-DD):
   ```bash
   grep "last_updated:" *.md | grep -E "[0-9]{4}-[0-9]{2}-[0-9]{2}"
   ```

3. **Verify status values** (lowercase):
   ```bash
   grep "status:" *.md | grep -v "status: \(draft\|review\|authoritative\|locked\|deprecated\)"
   ```

4. **Confirm commit success**:
   ```bash
   git log -1 --oneline | grep "frontmatter"
   ```

---

## Examples by Document Type

### Agent Profile Example
```markdown
<!--
AGENT_READ_FIRST (MUST)
- Read this entire document top-to-bottom before responding.
- Treat this file as the authoritative behavior contract.
-->

<!--
HAL9000_ENTRYPOINT
Agent: hal9000
Version: v5.2
Mode: Execution Orchestrator
Behavior: Planning-then-Execution
Start State: Planning Phase
Authority: Execution
Status: Authoritative
Last Updated: 2026-02-08
-->

# HAL9000 — Operating Protocol
```

### Playbook Example
```markdown
---
type: playbook
purpose: Initialize new GitHub projects with standardized structure and security
version: 1.0
status: authoritative
last_updated: 2026-02-08
tags: [github, setup, playbook, initialization]
---

# GitHub Project Setup Playbook

> **Purpose**: Initialize new GitHub projects with standardized structure and security
> **Version**: 1.0
> **Status**: Authoritative
> **Last Updated**: 2026-02-08

---
```

### Standard/Profile Example
```markdown
# Developer Standards Profile
# Version: v3.2
# Status: Locked
# Scope: Development standards, agent coordination, execution discipline
# Last Updated: 2026-02-08

---
```

### Template Example
```markdown
---
type: template
use_for: Documenting REST APIs with OpenAPI-compatible structure
version: 1.0
last_updated: 2026-02-08
tags: [template, api, openapi]
---

# API Specification Template

> **Type**: Template
> **Use For**: Documenting REST APIs with OpenAPI-compatible structure
> **Version**: 1.0

---
```

---

## Invariants

1. **YAML frontmatter MUST use `---` delimiters** for machine-parseability
2. **Status values MUST be lowercase** (draft, review, authoritative, locked, deprecated)
3. **Dates MUST use ISO format** (YYYY-MM-DD)
4. **Agent profiles MUST use HTML comments** for AI instruction clarity
5. **Playbooks MUST include purpose statement** in both YAML and blockquote
6. **Version numbers MUST be semantic** (major.minor or vmajor.minor)
7. **Tags MUST be array format** in YAML (`tags: [tag1, tag2]`)
8. **Type field MUST match document category** (playbook, guide, template, etc.)

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-02-08 | Initial frontmatter playbook consolidating standards, examples, and file type profiles into single authoritative source |

---

*Metadata Architect — Making documents discoverable, parseable, and maintainable.*
