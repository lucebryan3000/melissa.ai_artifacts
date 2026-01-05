# Script Lint Playbook
ID: SLP-001  
Version: 1.2  
Status: Authoritative  
Applies To: ops-script-library  
Audience: Humans and AI systems (Melissa.ai)

---

## 1. Purpose

This playbook defines the mandatory linting phase for all scripts created under the Script Creator Playbook.

Linting exists to enforce:
- Consistency
- Safety
- Automation readiness
- Compliance with declared scope and intent

A script MUST NOT proceed to review, packaging, or automation unless this playbook completes successfully.

---

## 2. When This Playbook Runs (AUTOMATIC)

This playbook MUST be invoked:

- Immediately after **Script Creation (Mode B)** completes
- Before **Script Reviewer Playbook** is invoked
- Without requiring user confirmation

Linting is NOT optional.  
Skipping linting is a process violation.

---

## 3. Linting Principles

Linting MUST be:

- Deterministic
- Non-interactive
- Fully automated
- Auditable

Linting MAY:
- Automatically fix safe, mechanical issues

Linting MUST NOT:
- Change script behavior
- Change script scope
- Introduce or remove logic
- Prompt the user
- Require human-in-the-loop interaction

---

## 4. Mandatory Lint Domains

Every lint run MUST evaluate the script against the following domains.

### 4.1 Standardized Frontmatter (REQUIRED)

Every script MUST include standardized comment frontmatter containing, at minimum:

- Synopsis
- Description
- Script type
- Scope (ReadOnly / Write / Destructive)
- Target platform
- OS support
- Runtime requirements
- Parameters
- Environment variables (if applicable)
- Usage examples
- `--help` / `-Help` behavior

Missing or incomplete frontmatter is a **blocking failure**.

---

### 4.2 Help & Usage Support

Scripts MUST:
- Expose `--help` or `-Help`
- Render usage and examples without execution
- Avoid undocumented parameters

Missing help support is a **blocking failure**.

---

### 4.3 Scope Accuracy

Lint MUST validate that the script’s behavior matches its declared scope.

Rules:
- ReadOnly scripts MUST NOT perform write or destructive actions
- Write scripts MUST declare where write behavior begins
- Destructive scripts MUST be explicitly classified

Scope mismatch is a **blocking failure**.

---

### 4.4 Determinism & CI Safety

Scripts MUST:
- Avoid interactive prompts by default
- Avoid formatted output (`Format-*`)
- Produce structured output
- Fail fast with clear errors
- Exit non-zero on fatal errors

Violations are **blocking failures**.

---

### 4.5 Error Handling

Scripts SHOULD include:
- Explicit error handling strategy
- Clear error messages
- Explicit exit behavior

Missing error handling is a **warning**, unless it creates silent failure paths.

---

### 4.6 Human-in-the-Loop (HITL) Compliance

Linting MUST ensure:
- No interactive prompts exist unless gated behind explicit flags
- Automation-safe defaults are preserved

Linting itself:
- MUST NEVER prompt
- MUST NEVER pause execution

Interactive behavior without gating is a **blocking failure**.

---

## 5. Lint Issue Classification

Every lint finding MUST be classified as one of the following.

### 5.1 Auto-Fixable (SAFE)

Issues that may be fixed automatically without changing semantics:

- Whitespace normalization
- Trailing newline insertion
- Comment formatting
- Frontmatter ordering or formatting
- Minor help text formatting
- Consistent casing
- Explicit exit code normalization

These MUST be fixed automatically.

---

### 5.2 Advisory (WARN)

Issues that do not block approval but should be reviewed:

- Optional guardrails
- Logging verbosity
- Defensive checks that could be added
- Style consistency improvements

Warnings MUST be reported but MUST NOT block progress.

---

### 5.3 Blocking (FAIL)

Issues that prevent the script from advancing:

- Missing standardized frontmatter
- Missing help support
- Scope misclassification
- ReadOnly script performing write actions
- Undeclared side effects
- Interactive prompts without flags
- Non-deterministic output
- Unsafe secret handling
- Deprecated tooling usage

Blocking issues MUST be resolved before proceeding.

---

## 6. Lint Execution Flow (MANDATORY)

Linting follows this exact sequence:

### Step 1 — Intake
- Confirm script creation phase completed
- Confirm script identity and declared scope

Failure here is terminal.

---

### Step 2 — Static Analysis
- Evaluate script against all mandatory lint domains
- Record all findings

---

### Step 3 — Auto-Fix Pass
- Automatically apply all SAFE auto-fixable changes
- Record each fix applied
- Do NOT apply fixes that would change behavior or scope

---

### Step 4 — Re-Lint
- Re-run full lint after auto-fixes
- Confirm no new issues were introduced

If blocking issues remain → FAIL.

---

### Step 5 — Verdict
Produce exactly one verdict:

- PASS
- PASS WITH WARNINGS
- FAIL

---

## 7. Lint Report (REQUIRED ARTIFACT)

Every lint run MUST produce a report containing:

- Script name
- Timestamp
- Auto-fixes applied
- Warnings
- Blocking issues
- Final verdict

This report MUST be available to the reviewer.

---

## 8. Automatic Handoff

If verdict is:
- PASS
- PASS WITH WARNINGS

Then the process MUST automatically proceed to:

➡️ **Script Reviewer Playbook**

No user confirmation required.

If verdict is FAIL:
- The process MUST stop
- Blocking issues MUST be explained

---

## 9. Enforcement

Scripts that:
- Skip linting
- Ignore blocking lint failures
- Proceed to review without a lint verdict

Are NON-COMPLIANT and MUST be rejected.

---

## 10. Definition of Done (Lint Phase)

The lint phase is complete only when:

- Lint ran automatically
- Safe auto-fixes were applied
- Re-lint succeeded
- Verdict recorded
- Handoff decision made

Anything less is NOT DONE.

---

End of Playbook
