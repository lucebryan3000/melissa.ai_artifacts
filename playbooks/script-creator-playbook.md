# Script Creator Playbook
ID: SCP-001  
Version: 1.3  
Status: Authoritative  
Applies To: ops-script-library  
Audience: Humans and AI systems (Melissa.ai)

---

## 1. Purpose

This playbook defines the mandatory process for creating operational scripts that are:

- Safe to automate
- Deterministic
- Auditable
- Human-reviewable
- Compatible with AI-assisted execution
- Suitable for shared internal libraries

Any script that does not fully comply with this playbook MUST NOT be published, automated, or reused.

---

## 2. Core Principles (Non-Negotiable)

All scripts MUST:

1. Be single-purpose
2. Be non-interactive by default
3. Fail fast with actionable errors
4. Produce deterministic output
5. Declare scope honestly (ReadOnly vs Write)
6. Separate authoring, linting, and review
7. Be safe for CI and unattended execution
8. Make side effects explicit
9. Support human-in-the-loop (HITL) when appropriate
10. Prefer clarity over cleverness

Violation of any principle is a defect.

---

## 3. Lifecycle Overview

Scripts progress through explicit, gated plays:

Mode A → Mode B → Mode C → Mode D  
Pre-Creation → Creation → Lint → Review

Optional post-approval plays:
- Packaging
- Orchestration
- AI augmentation

No play may be skipped.

---

## 4. Mode A — Pre-Creation Context Lock (MANDATORY)

### 4.1 Script Identity

Every script MUST define:

- **Package ID**  
  Format: `domain.script-name` (kebab-case, immutable)

- **Human Name**  
  Imperative verb phrase describing exactly what the script does

Example:
```

Package ID: azure.pull-graph-data
Human Name: Pull Microsoft Graph data from Azure AD

````

#### Options
1. **Single, narrowly scoped script** (recommended)  
   _Reason: Easier to review, lint, automate, and reason about_
2. Multi-purpose script
3. Script that delegates to other scripts

---

### 4.2 Script Type & Scope Declaration (NEW)

Every script MUST declare:

- Script type:
  - Operational
  - Validation
  - Reporting
  - Orchestration
- Scope classification:
  - ReadOnly
  - Write
  - Destructive

Rules:
- Scope MUST be accurate
- Transitioning from ReadOnly → Write requires a **new script**
- Destructive scripts require explicit human confirmation

#### Options
1. **ReadOnly script** (recommended)  
   _Reason: Safest default, easiest to automate_
2. Write script with safeguards
3. Destructive script with mandatory HITL

---

### 4.3 Target & OS Declaration

Choose exactly one target:
- azure
- aws
- windows
- macos
- ubuntu
- sql

Declare OS support:
```yaml
os:
  primary: windows | macos | ubuntu
  compatible:
    - optional
````

#### Options

1. **Single primary OS, optional compatible OS** (recommended)
2. Multi-OS primary support
3. OS-agnostic (rare, must justify)

---

### 4.4 Shell & Runtime

Choose exactly one:

* powershell
* bash
* sql

Declare minimum runtime version.

#### Options

1. **Modern, supported runtime** (recommended)
2. Legacy runtime (discouraged)
3. Multiple runtimes (requires justification)

---

### 4.5 Tooling Contract

Declare:

* Required tools
* Minimum versions

Rules:

* No deprecated modules
* No floating versions
* Fail fast if unmet

#### Options

1. **Minimal, explicit tooling** (recommended)
2. Broad tooling set
3. Tool detection at runtime

---

### 4.6 Environment Contract

Declare:

* Required environment variables
* Optional environment variables
* DRY_RUN support

Rules:

* Secrets via environment variables or managed identity only
* No prompting for secrets
* DRY_RUN disables all external side effects

#### Options

1. **Env vars + DRY_RUN supported** (recommended)
2. Env vars only
3. Managed identity only

---

### Proceed to Next Play

➡️ **Prompt:** Run Script Creation (Mode B)
➡️ Next recommended play after completion: **Script Lint Playbook**

---

## 5. Mode B — Script Creation

### 5.1 Standardized Frontmatter (MANDATORY, NEW)

Every script MUST include standardized frontmatter, regardless of type:

Frontmatter MUST declare:

* Synopsis
* Description
* Script type
* Scope (ReadOnly / Write / Destructive)
* Target platform
* OS support
* Runtime requirements
* Parameters
* Environment variables
* Usage examples
* `--help` / `-Help` behavior

This is REQUIRED for **all scripts**, not optional.

---

### 5.2 Determinism & Output Rules

Scripts MUST:

* Avoid interactive prompts by default
* Return structured data (objects, JSON)
* Avoid formatted output
* Emit timestamps explicitly
* Clearly declare side effects

---

### 5.3 Error Handling (ENHANCED)

Scripts MUST:

* Validate inputs early
* Validate environment before execution
* Fail fast with clear messages
* Exit non-zero on fatal errors
* Never silently continue

---

### 5.4 Human-in-the-Loop (HITL) Rules (ENHANCED)

Scripts MAY prompt ONLY IF:

* An explicit `-Confirm` or `-Interactive` flag is passed
* Execution context is interactive (TTY detected)
* Automation mode never blocks

Required HITL pattern:

1. Describe intended action
2. Display scope and impact
3. Require explicit confirmation
4. Abort safely on denial or timeout

---

### 5.5 ReadOnly vs Write Guardrails (NEW)

Rules:

* ReadOnly scripts MUST NOT mutate external systems
* Write scripts MUST:

  * Declare changes clearly
  * Support DRY_RUN where feasible
  * Escalate to HITL if risk is high
* Any Write action MUST be obvious from the script name and frontmatter

---

### Proceed to Next Play

➡️ **Prompt:** Run Script Lint Playbook
➡️ Next recommended play after lint: **Script Reviewer Playbook**

---

## 6. Mode C — Script Lint (REQUIRED)

Linting MUST:

* Be explicitly executed
* Follow Script Lint Playbook
* Produce PASS / WARN / FAIL
* Record findings and remediation

Manual best practices do NOT count as linting.

#### Options

1. **Automated lint with recorded output** (recommended)
2. Manual lint checklist
3. No lint (NOT allowed)

---

## 7. Mode D — Script Review (REQUIRED)

Review MUST:

* Be performed by a different role than author
* Follow Script Reviewer Playbook
* Produce one verdict:

  * APPROVED
  * APPROVED WITH NOTES
  * REJECTED

No verdict = not approved.

#### Options

1. **Independent reviewer with checklist** (recommended)
2. Pair review
3. Self-review (NOT allowed)

---

## 8. AI Compatibility (Clarified)

Scripts MAY include Melissa execution markers, but they are OPTIONAL.

Markers:

* Are comments only
* Must not affect runtime
* Enable pause, resume, and policy enforcement

Scripts are NOT required to include markers to be compliant.

---

## 9. Definition of Done (ENHANCED)

A script is DONE only when:

* Pre-creation context is locked
* Script is created with standardized frontmatter
* Script passes lint
* Script is reviewed
* Verdict is recorded
* HITL behavior is documented
* Scope classification is accurate

Anything less is WORK IN PROGRESS.

---

## 10. Enforcement

Non-compliant scripts:

* MUST NOT be merged
* MUST NOT be automated
* MUST NOT be published

This playbook exists to prevent:

* Hero scripts
* Silent failures
* Unsafe automation
* Unreviewed operational risk

Compliance is mandatory.

---

## Recommended Next Steps (Consistent Pattern)

1. **Run Script Lint Playbook** (recommended first)
   *Why: catches defects before human review*

2. **Run Script Reviewer Playbook**
   *Why: formal approval and risk acknowledgment*

3. **Proceed to Packaging or Orchestration**
   *Why: only approved scripts should be automated*

---

End of Playbook


Say which one.
```
