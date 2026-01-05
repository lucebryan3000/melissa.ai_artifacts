# Script Lint Playbook  
**Playbook ID:** SLP-001  
**Name:** Script Lint Playbook  
**Version:** 1.0  
**Status:** Canonical  
**Classification:** **Evaluative (Static Analysis)**  
**Applies To:** All scripts in `ops-script-library`  
**Generated Using:** playbook-generator-playbook.md  
**Primary Consumer:** AI systems, optional CI tooling, human reviewers

---

## 1. Playbook Role & Authority

### 1.1 What This Playbook Governs

This playbook defines **static linting rules** for scripts.

It governs:
- Style correctness
- Portability red flags
- Dangerous patterns
- Consistency with declared intent

Linting is **advisory unless explicitly marked blocking**.

---

### 1.2 Relationship to Other Playbooks

Hierarchy:

Script Creator Playbook (authority)
└── Script Reviewer Playbook (merge gate)
└── Script Lint Playbook (signal)

yaml
Copy code

Linting informs decisions but does not override reviewer judgment.

---

## 2. Invocation Modes

### Mode A — Author Self-Lint (RECOMMENDED)

Used by authors or AI before submission.

---

### Mode B — Reviewer Lint (RECOMMENDED)

Used during review to surface risks.

---

### Mode C — CI Lint (OPTIONAL)

Used to automate detection of known issues.

---

## 3. General Lint Rules (All Scripts)

Lint warnings should be raised for:

- Unquoted variables
- Silent error suppression
- Hard-coded paths
- Implicit defaults
- Unused environment variables

---

## 4. Bash-Specific Lint Rules

### 4.1 Blocking Issues

- Missing `#!/usr/bin/env bash`
- Missing `set -euo pipefail`
- Use of `sed -i` without OS handling
- Unquoted `$VAR` usage
- Backticks instead of `$(...)`

Blocking issues must be fixed.

---

### 4.2 Advisory Issues

- `awk`, `grep`, `sed` portability risks
- Use of `[[` vs `[`
- Non-POSIX flags

Advisory issues should be documented or justified.

---

## 5. PowerShell-Specific Lint Rules

### 5.1 Blocking Issues

- Windows-only APIs in non-Windows scripts
- Hard-coded paths
- Missing `Set-StrictMode` (recommended)

---

### 5.2 Advisory Issues

- Use of aliases
- Unscoped variables
- Excessive inline logic

---

## 6. Cloud Script Lint Rules

Raise warnings if:

- Account / subscription not echoed
- Region not displayed
- Destructive commands lack guardrails

---

## 7. SQL Script Lint Rules

Blocking:
- Write statements in `ReadOnly` scripts

Advisory:
- Unbounded queries
- Missing comments
- Lack of ordering

---

## 8. Lint Output Format

Lint tools (human or AI) should emit:

- Rule ID
- Severity (Blocking | Advisory)
- File
- Line (if applicable)
- Explanation

---

## 9. Failure Semantics

- Blocking lint issues → must be fixed
- Advisory issues → reviewer discretion

Linting **never auto-approves** scripts.

---

## 10. What This Playbook Does NOT Do

- Does not execute scripts
- Does not replace review
- Does not auto-fix code
- Does not enforce style preferences

---

## 11. Versioning

- This playbook is versioned
- Lint rules may expand
- Breaking changes require major bump

---