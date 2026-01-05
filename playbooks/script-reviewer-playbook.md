# Script Reviewer Playbook
ID: SRP-001  
Version: 1.3  
Status: Authoritative (Advisory / Guardrails)  
Applies To: ops-script-library  
Audience: Humans and AI systems (Melissa.ai)

---

## 1. Purpose

This playbook defines the **recommended review process** for scripts created under the Script Creator Playbook and validated by the Script Lint Playbook.

The reviewer’s role is to:
- Validate intent, scope, and safety
- Surface risks and mismatches
- Provide an explicit recommendation before reuse or automation

This playbook emphasizes **guardrails and visibility**, not hard blocking.

---

## 2. When This Playbook Runs

This playbook SHOULD be invoked:

- After the Script Lint Playbook completes
- Regardless of whether lint produced warnings
- Before packaging, orchestration, or automation

Review is strongly recommended for **all scripts**, and especially critical for:
- Write or Destructive scripts
- Scripts intended for automation or CI
- Scripts touching production systems

---

## 3. Reviewer Role & Expectations

The reviewer SHOULD:

- Be logically independent from the script author when possible
- Review the script as if it could be run unattended
- Focus on risk, clarity, and correctness—not style preferences
- Assume future readers and operators are unfamiliar with the author’s intent

For AI reviewers:
- Explicitly switch context from “author” to “reviewer”
- Prefer conservative interpretations of risk

Self-review is discouraged but permitted when no alternative exists (must be noted).

---

## 4. Reviewer Inputs (Recommended)

The reviewer SHOULD have access to:

1. The final script
2. Standardized frontmatter
3. Lint report (including warnings and auto-fixes)
4. Pre-creation context (intent, scope, target)
5. Declared script type and scope

If any input is missing, the reviewer SHOULD note this as a risk.

---

## 5. Review Philosophy (Guardrails)

Review SHOULD be:

- Explicit
- Auditable
- Deterministic
- Non-interactive

Review SHOULD NOT:
- Modify code directly
- Perform linting
- Auto-fix issues

Review MAY:
- Recommend changes
- Flag risks
- Suggest follow-up actions
- Recommend escalation for high-risk scripts

---

## 6. Recommended Review Checklist (WITH GUIDANCE)

The reviewer SHOULD walk through each section and record observations.

### 6.1 Identity & Intent

Check that:
- Script purpose matches its Human Name
- Script does what it claims—and only that
- No hidden responsibilities exist

**Enhancement #1:**  
Reviewer explicitly states whether intent is *clear*, *mostly clear*, or *ambiguous*.

---

### 6.2 Frontmatter & Documentation

Verify that:
- Standardized frontmatter exists
- Script type is declared
- Scope (ReadOnly / Write / Destructive) is declared
- OS, runtime, and tooling are documented
- Usage examples appear correct
- `--help` / `-Help` renders safely

**Enhancement #2:**  
Reviewer flags missing or unclear documentation even if lint passed.

---

### 6.3 Scope Accuracy (READ vs WRITE)

Confirm that:
- ReadOnly scripts do not mutate external systems
- Write scripts clearly mark where changes occur
- Destructive scripts are explicitly labeled

**Enhancement #3:**  
Reviewer explicitly notes where the script transitions from ReadOnly → Write (if applicable).

---

### 6.4 Lint Findings Awareness

Reviewer SHOULD:
- Confirm lint ran
- Review warnings
- Decide whether warnings are acceptable given context

**Enhancement #4:**  
Reviewer records whether lint warnings are:
- Accepted
- Deferred
- Recommended for remediation

---

### 6.5 Error Handling & Failure Modes

Check for:
- Early validation
- Clear error messages
- No silent failures
- Predictable exit behavior

**Enhancement #5:**  
Reviewer explicitly states worst-case failure impact.

---

### 6.6 Determinism & Automation Safety

Confirm that:
- Script is non-interactive by default
- Automation will not hang
- Output is structured and predictable

**Enhancement #6:**  
Reviewer notes whether script is safe for:
- Manual use
- CI
- Scheduled automation

---

### 6.7 Human-in-the-Loop (HITL) Behavior

If HITL exists, verify:
- It is opt-in
- Prompts explain impact
- Refusal exits safely

**Enhancement #7:**  
Reviewer explicitly notes whether HITL behavior is:
- Required
- Optional
- Not applicable

---

### 6.8 Security & Secrets

Check that:
- Secrets are not hard-coded
- Secrets are not logged
- Environment variables or managed identity are used

**Enhancement #8:**  
Reviewer flags any area where secret handling *might* become risky over time.

---

### 6.9 Operational Risk & Blast Radius

Assess:
- Potential impact if script misbehaves
- Whether rollback or recovery is obvious
- Whether script should be restricted to certain environments

**Enhancement #9:**  
Reviewer classifies risk as:
- Low
- Medium
- High

---

## 7. Reviewer Recommendation (NOT A BLOCK)

Instead of a hard verdict, the reviewer MUST provide **one recommendation**:

- **RECOMMENDED TO PROCEED**  
  Script is safe and clear for intended use.

- **PROCEED WITH CAUTION**  
  Script is usable, but risks or gaps exist (must be listed).

- **RECOMMEND REMEDIATION BEFORE USE**  
  Script should be improved before automation or reuse.

This recommendation is advisory but should be respected.

---

## 8. Review Notes & Report

The review SHOULD produce a short report including:

- Script name
- Reviewer identity
- Review timestamp
- Lint summary reference
- Risk classification
- Recommendation
- Notes and follow-ups (if any)

**Enhancement #10:**  
Reviewer explicitly states whether the script is suitable for automation.

---

## 9. Handoff Guidance

Based on recommendation:

- **Recommended to Proceed**  
  → Packaging, orchestration, or automation is reasonable

- **Proceed with Caution**  
  → Prefer limited rollout, documentation, or manual execution

- **Recommend Remediation**  
  → Address notes before reuse or automation

---

## 10. Definition of Done (Review Phase)

A review is considered complete when:

- Reviewer has examined the script
- Checklist areas are considered
- Recommendation is recorded
- Risks and notes are documented

Review does not guarantee correctness—it provides **informed confidence**.

---

End of Playbook
