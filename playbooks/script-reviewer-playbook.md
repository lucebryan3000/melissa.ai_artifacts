# Script Reviewer Playbook  
**Playbook ID:** SRP-001  
**Name:** Script Reviewer Playbook  
**Version:** 1.0  
**Status:** Canonical  
**Classification:** **Evaluative**  
**Applies To:** All scripts submitted to `ops-script-library`  
**Generated Using:** playbook-generator-playbook.md  
**Primary Consumer:** Human reviewers, AI reviewers (Melissa.ai)

---

## 1. Playbook Role & Authority

### 1.1 What This Playbook Governs

This playbook governs **how scripts are reviewed and approved** before entering the repository.

It defines:
- What reviewers must check
- What constitutes approval vs rejection
- When reviewers must block a script
- How reviewer judgment is applied consistently

If a script fails this playbook, **it must not be merged**, regardless of functionality.

---

### 1.2 Who Must Use This Playbook

This playbook **must** be used by:
- Human reviewers
- AI reviewers
- Maintainers approving merges

Skipping this playbook invalidates the review.

---

### 1.3 Authority Level

This playbook has **merge-blocking authority**.

Reviewers:
- Must approve or reject explicitly
- Must not “approve with comments”
- Must not defer required fixes

---

## 2. Invocation Modes

### Mode A — Initial Review (MANDATORY)

Used when a script is first submitted.

Purpose:
- Verify compliance with Script Creator Playbook
- Detect high-risk issues early

---

### Mode B — Re-Review (MANDATORY if changes requested)

Used after author revisions.

Purpose:
- Confirm issues were resolved
- Ensure no regressions were introduced

---

## 3. Reviewer Preconditions

Before reviewing, confirm:

- The **Script Creator Playbook v1.0** was followed
- The script declares:
  - Target
  - Primary OS
  - Scope
- Reviewer understands the script’s intent

If not, **stop and request clarification**.

---

## 4. Normative Review Rules

### 4.1 Completeness Rules

Reviewer must confirm:

- Package directory exists
- `package.yaml` is present and valid
- `.env.example` exists
- Entry script exists and matches manifest

Missing artifacts = automatic rejection.

---

### 4.2 Truthfulness Rules

Reviewer must validate:

- Primary OS matches script content
- Compatible OS claims are honest
- Scope classification matches behavior
- Tool requirements are complete

Any mismatch = rejection.

---

### 4.3 Safety Rules

Reviewer must verify:

- `ReadOnly` scripts do not mutate state
- `Destructive` scripts:
  - Clearly warn
  - Require confirmation
- No hard-coded credentials or IDs
- No hidden side effects

Safety violations = rejection.

---

### 4.4 Portability Rules

Reviewer must check:

- No OS-specific flags contradict declared OS
- Bash scripts avoid GNU/BSD traps or document them
- PowerShell scripts respect declared platform

Violations must be fixed or compatibility reduced.

---

## 5. Reviewer Checklist (MANDATORY)

All must be true to approve:

- [ ] Script matches declared target
- [ ] Primary OS is correct
- [ ] Scope classification is honest
- [ ] No secrets embedded
- [ ] `.env.example` is complete
- [ ] Tool dependencies declared
- [ ] Script runs without launcher
- [ ] Failure modes are understandable
- [ ] Destructive actions are gated
- [ ] Documentation is sufficient

If any box is unchecked → **reject**.

---

## 6. Reviewer Decision Outcomes

Reviewer must choose **exactly one**:

- **Approve** — script may merge
- **Reject** — script must be fixed

“Approve with comments” is not allowed.

---

## 7. What This Playbook Does NOT Do

- Does not rewrite scripts
- Does not test execution
- Does not waive requirements
- Does not optimize scripts

---

## 8. AI Reviewer Behavior

AI reviewers must:
- Apply the same checklist
- Cite specific violations
- Refuse approval on failures
- Avoid subjective opinions

---

## 9. Versioning

- This playbook is versioned
- Reviews should note version used
- Breaking changes require major bump

---

