# KB Validation Playbook - Complete Reference

> **Purpose**: Full validation protocol for KB articles
> **Version**: 2.0.0
> **Last Updated**: 2025-12-26
> **Framework**: Phase 18 KBGEN with config-driven scoring

---

## ⚠️ Note on Scoring System Versions

This playbook documents an **aspirational validation framework** (quality-thresholds-2026.yaml v2.3.0) that provides comprehensive scoring methodology.

**Phase 18 KBGEN Implementation** uses a different scoring model:
- **Location**: `src/kbgen/config.ts` (quality.scoringModel)
- **Factors**: 10-factor model matching this playbook's factors
- **Max Score**: 100 (normalized from sum of individual factors)
- **Configuration**: All thresholds and weights are centralized and environment-variable configurable

**For current KBGEN validation**, see:
- `src/kbgen/services/scorer.ts` - Actual 10-factor scoring implementation
- `.claude/commands/kbgen.md` - Configuration reference with current thresholds
  - Gap detection thresholds: clarity=60, completeness=60, codeQuality=50
  - Target quality score: 75 (configurable)
  - Max enhancement attempts: 2 (configurable)

This playbook serves as the **specification for optimal validation practices**. The Phase 18 implementation achieves these goals within its scoring framework.

---

## PASS 1: HAIKU VALIDATION (Fast Checks)

Execute with Haiku for cost-effective validation:

### Syntax Checks
- [ ] Valid markdown syntax
- [ ] YAML frontmatter present and parseable
- [ ] Code blocks properly fenced (opening and closing ```)
- [ ] All code blocks have language tags
- [ ] Headers use proper markdown (##, ###)
- [ ] No broken links in markdown

### Structure Checks
- [ ] Required frontmatter fields: `title`, `description`, `tags`, `author`, `date`, `article_type`
- [ ] Numbered H2 sections (## 1., ## 2., etc.)
- [ ] Minimum section count met (Reference: 8, Tutorial: 6, Troubleshooting: 4, Quickstart: 4)
- [ ] Summary section present
- [ ] Next Steps section present

### Placeholder Detection
- [ ] No TODO markers
- [ ] No FIXME comments
- [ ] No `[X]` placeholders
- [ ] No `{{variable}}` unreplaced
- [ ] No `...` ellipsis in code
- [ ] No "Add content here" phrases

### Code Syntax Validation (+2 bonus potential)
Extract all code blocks and validate syntax:

```bash
# TypeScript validation
npx tsc --noEmit {temp_file}.ts

# Python validation
python -m ast {temp_file}.py

# JavaScript validation
npx eslint --no-eslintrc {temp_file}.js
```

**Haiku Output Format**:
```
PASS 1: HAIKU VALIDATION
========================

✓ Markdown syntax valid
✓ YAML frontmatter present
✓ 18 code blocks, all properly fenced
✓ All code blocks have language tags
✓ Header structure valid (8 numbered H2s)
✓ No placeholders detected
✓ Syntax validation: 18/18 blocks pass TypeScript check

STRUCTURE:
- Article type: reference
- Sections: 8/8 required
- Word count: 3247 words
- Code blocks: 18 examples

RESULT: ✅ PASS 1 COMPLETE - Ready for scoring
```

**If Validation Fails**:
```
PASS 1: HAIKU VALIDATION
========================

✗ VALIDATION FAILED

Issues found:
1. Missing closing fence on line 342 (code block not closed)
2. Placeholder detected: "TODO: Add rate limiting example" (line 567)
3. Code block missing language tag (line 123)
4. Syntax error: TypeScript block at line 234 fails validation
   Error: Type 'string' is not assignable to type 'number'

RESULT: ❌ FAIL - Fix issues before scoring
```

---

## PASS 2: SONNET SCORING (10-Factor Quality)

Execute with Sonnet for comprehensive scoring:

### 10-Factor Scoring (quality-thresholds-2026.yaml v2.3.0)

**Scoring Formula**:
```
base_score = (
  (code_examples * 0.20) +           # 20%
  (type_safety * 0.15) +             # 15%
  (error_handling * 0.10) +          # 10%
  (section_completeness * 0.08) +    # 8%
  (header_structure * 0.08) +        # 8%
  (terminology_consistency * 0.04) + # 4%
  (production_patterns * 0.10) +     # 10%
  (copy_paste_ready * 0.10) +        # 10%
  (standalone_completeness * 0.05) + # 5%
  (security_best_practices * 0.05)   # 5%
) * 10

total_score = base_score + bonuses + deductions
```

### Factor Analysis

| Factor | Weight | Target | Description |
|--------|--------|--------|-------------|
| 1. Code Examples | 20% | 10/10 | Complete implementations, 15+ examples |
| 2. Type Safety | 15% | 10/10 | Strict mode, generics, no 'any' |
| 3. Error Handling | 10% | 10/10 | Custom errors, try/catch, recovery |
| 4. Section Completeness | 8% | 10/10 | All sections, 200-500 words each |
| 5. Header Structure | 8% | 10/10 | Numbered H2s, pattern names |
| 6. Terminology | 4% | 10/10 | Consistent terms, no drift |
| 7. Production Patterns | 10% | 9/10 | Logging, monitoring, cleanup |
| 8. Copy-Paste Ready | 10% | 10/10 | Imports shown, runnable |
| 9. Standalone | 5% | 10/10 | Self-contained sections |
| 10. Security | 5% | 8/10 | Validation, auth, encryption |

### Bonuses (Max +9 Points)

| Bonus | Points | Criteria |
|-------|--------|----------|
| Cross-References | +2 | 4+ quality internal KB links |
| Vendor Docs | +2 | 2+ official documentation URLs |
| Interactive Content | +3 | 2+ CodeSandbox/StackBlitz demos |
| Syntax Validation | +2 | All code blocks pass validation |

### Deductions (Max -45 Points)

**Security Issues**:
- Hardcoded secrets: -10
- SQL injection risk: -5
- XSS vulnerability: -5
- Weak auth patterns: -3

**Code Quality**:
- Broken examples: -5
- Incomplete imports: -3
- Missing error handling: -3
- Anti-patterns: -5

**Freshness**:
- 181-365 days old: -5
- 366+ days old: -10

---

## PASS 3: AUTHORITATIVE SOURCE VALIDATION

Execute with Sonnet for verification against official documentation.

**When to Use Pass 3**:
- Articles claiming specific API versions
- Articles with code examples from external libraries/frameworks
- Articles with performance claims or configuration recommendations
- Articles referencing official documentation URLs

### Validation Checks

**API Version Verification**:
- Fetch official package repository (npm, PyPI, etc.)
- Verify claimed versions are current or clearly outdated
- Check for breaking changes in claimed version ranges
- Document version constraints in article

**Official Documentation Alignment**:
- Fetch official docs for referenced libraries/frameworks
- Verify code examples match current API signatures
- Check if deprecated patterns are clearly marked
- Validate configuration recommendations

**Freshness Assessment**:
- Compare article date with official docs last update
- Identify outdated content requiring refresh
- Flag if SDK/library major version has changed since article creation

**Authoritative Source Checks**:
- Fetch official vendor documentation URLs
- Verify claims against authoritative sources (not blog posts/tutorials)
- Check for contradictions with official best practices
- Validate security recommendations against current standards

### Pass 3 Output Format

```
PASS 3: AUTHORITATIVE SOURCE VALIDATION
========================================

ARTICLE: anthropic-sdk-typescript/07-TOKEN-MANAGEMENT.md
FRAMEWORK: Anthropic SDK TypeScript

VERSION VERIFICATION
---------------------
Claimed Version: @anthropic-ai/sdk 0.27.3
Official Latest: 0.28.0
Status: ⚠️ OUTDATED (3 releases behind)
Risk Level: LOW (minor/patch updates only)

API SIGNATURE VALIDATION
------------------------
✓ messages.create() - Signature matches docs
✓ prompt caching pattern - Documented in official examples
✓ streaming with .stream() - Current recommended approach

FRESHNESS ASSESSMENT
--------------------
Article Created: 2025-11-13
SDK Last Updated: 2025-12-20 (37 days newer)
Freshness Score: 92%

OVERALL PASS 3 RESULT
=====================
Status: ✅ APPROVED WITH NOTES
Credibility: 98%
```

### Deductions for Outdated Information

| Severity | Deduction | Criteria |
|----------|-----------|----------|
| Critical | -15 | Major version mismatch, invalid API, security vulnerability |
| High | -10 | 2+ versions behind, deprecated methods, undocumented breaking change |
| Medium | -5 | Patch outdated, deprecated but working, better approach available |
| Low | -2 | Within 1 version, no functional impact |

---

## TIER ASSIGNMENT

| Tier | Range | Icon | Action |
|------|-------|------|--------|
| **Exemplary** | 90-109 | 🏆 | Use as template for new articles |
| **Target Met** | 80-89 | ✅ | Approved for production use |
| **Acceptable** | 70-79 | ⚠️ | Schedule improvement pass |
| **Needs Work** | 60-69 | 🔧 | Prioritize for rewrite |
| **Critical** | 0-59 | ❌ | Immediate manual review required |

---

## VALIDATION CAPABILITIES

### What This Validates ✅

**Pass 1-2 (Always)**:
- Markdown syntax, YAML frontmatter
- Code block structure and syntax
- Placeholder detection
- 10-factor quality scoring
- Production readiness

**Pass 3 (Optional with `--verify-sources`)**:
- API version verification
- Official documentation alignment
- Authoritative source validation
- Freshness assessment

### What This Does NOT Do ❌

- Run/execute code examples
- Test performance claims
- Deploy code
- Compare against other KB articles

---

## REFERENCES

**Playbooks**:
- [Phase-12.1-KBGEN.md](_build/prompts/Phase-12.1-KBGEN.md) - Orchestration & quality control
- [Phase-12.2-KBGEN.md](_build/prompts/Phase-12.2-KBGEN.md) - Codex templates

**Scoring Specification**:
- [quality-thresholds-2026.yaml](_build/prompts/kb/quality-thresholds-2026.yaml) - v2.3.0 authoritative source

**Model Strategy**:
- Haiku: Fast validation (~$0.01/article)
- Sonnet: Quality scoring (~$0.50/article)
- Sonnet: Source validation (~$1.00/article)
