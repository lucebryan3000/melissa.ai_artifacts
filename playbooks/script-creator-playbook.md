# Script Creator Playbook

**Playbook ID:** SCP-001
**Name:** Script Creator Playbook
**Version:** 1.0
**Status:** Canonical
**Classification:** **Hybrid (Generative + Evaluative)**
**Applies To:** `ops-script-library`
**Generated Using:** playbook-generator-playbook.md
**Primary Consumer:** Humans and AI systems (Melissa.ai)

---

## 1. Playbook Role & Authority

### 1.1 What This Playbook Governs

This playbook defines **how scripts are created, structured, validated, and approved** for inclusion in the Ops Script Manager repository.

It governs:

* Script intent definition
* OS targeting and compatibility
* Safety classification
* Frontmatter / manifest requirements
* Authoring constraints
* Validation and approval criteria

If a script conflicts with this playbook, **the script is invalid**, regardless of whether it “works.”

---

### 1.2 Who Must Use This Playbook

This playbook **must** be used by:

* Any human author creating scripts for the repo
* Any AI system (including Melissa.ai / ChatGPT) generating scripts
* Any reviewer approving scripts for merge

Use is **mandatory**, not advisory.

---

### 1.3 Authority Level

This playbook has **normative authority**.

* It may **block creation**
* It may **invalidate scripts**
* It may **require regeneration**
* It may **refuse output** if constraints are violated

---

## 2. Invocation Modes (Required by Generator Framework)

This playbook is invoked in **three explicit modes**.

### Mode A — Pre-Creation (Generative, Mandatory)

**Invocation phrase (AI):**

> “Use the Script Creator Playbook — Pre-Creation”

Purpose:

* Lock intent
* Lock OS assumptions
* Lock safety scope
* Prevent rework

**No code may be written before this mode completes.**

---

### Mode B — In-Flight Validation (Evaluative, Optional but Recommended)

**Invocation phrase (AI):**

> “Validate against the Script Creator Playbook — In-Flight”

Purpose:

* Detect drift
* Catch portability and safety violations early
* Correct patterns before they harden

---

### Mode C — Final Health Check (Evaluative, Mandatory)

**Invocation phrase (AI):**

> “Run Script Creator Playbook — Final Health Check”

Purpose:

* Certify readiness
* Approve or reject the script
* Decide whether the script may enter the repo

---

## 3. Normative Rules (Non-Negotiable)

### 3.1 Package-First Rule

Every script **must** be delivered as a **package**, not a loose file.

A package is:

* A directory
* With a manifest
* With a single entrypoint
* With an explicit env contract

---

### 3.2 Explicit OS Rule

Scripts **must** declare the OS they were written for.

Allowed OS values:

* `windows`
* `ubuntu`
* `macos`

There is **no “POSIX”** classification in authoring language.

---

### 3.3 Truthful Compatibility Rule

If a script:

* Has not been tested on an OS
* Uses OS-specific flags or tools

Then that OS **must not** be declared as primary.

Compatibility lists mean:

> “Likely works, user assumes risk.”

---

### 3.4 Launcher Independence Rule

Scripts **must run without the launcher**.

The launcher:

* Orchestrates
* Does not provide logic
* Does not fix broken scripts

---

## 4. Pre-Creation Context Lock (Mode A)

All items below **must be answered before code is written**.

### 4.1 Identity

* **Package ID**
  Format: `<target>.<short-name>`
  Example: `azure.list-vms`

* **Human Name**
  Clear, imperative
  Example: `List Azure Virtual Machines`

---

### 4.2 Target Declaration

Choose **exactly one** target:

* `windows`
* `azure`
* `aws`
* `sql`
* `macos`
* `ubuntu`

---

### 4.3 OS Declaration

```yaml
os:
  primary: ubuntu | macos | windows
  compatible:
    - optional
```

Primary OS = written and tested for.
Compatible OSes = user may attempt at own risk.

---

### 4.4 Shell & Runtime

Choose **exactly one**:

* `powershell`
* `bash`
* `sql` (queries only)

Windows PowerShell is **Windows-only**.
Bash and cross-platform PowerShell must assume Unix semantics.

---

### 4.5 Scope Classification

Choose **one and only one**:

* `ReadOnly`
* `Write`
* `Destructive`

Misclassification is a defect.

---

### 4.6 Tooling Contract

Explicitly list:

* Required tools
* Minimum versions (if relevant)

If it’s not declared, it cannot be assumed.

---

### 4.7 Environment Contract

Define:

* Required env vars
* Optional env vars
* Whether `DRY_RUN` is supported

---

## 5. Required Package Artifacts

### 5.1 Mandatory Files

```
<package>/
├── run.sh | run.ps1
├── package.yaml
└── .env.example
```

Missing files = invalid package.

---

### 5.2 `package.yaml` — Required Fields

```yaml
schemaVersion: 1
id: azure.list-vms
name: List Azure VMs
target: azure

os:
  primary: ubuntu
  compatible:
    - macos

entry:
  type: bash
  file: run.sh

scope: ReadOnly
owner: ops
```

---

### 5.3 `.env.example` Rules

* No secrets
* No real IDs
* Comments explain intent
* Defaults only if safe

---

## 6. Authoring Constraints (Mode B)

### 6.1 Bash Scripts (Ubuntu / macOS)

**Must:**

* Use `#!/usr/bin/env bash`
* Use `set -euo pipefail`
* Quote variables
* Fail fast

**Must Not:**

* Assume GNU vs BSD parity
* Use OS-specific paths silently
* Write outside working directory

---

### 6.2 Cross-Platform PowerShell

**Must:**

* Target PowerShell 7+
* Avoid registry, COM, Win32
* Assume case-sensitive FS

---

### 6.3 Windows PowerShell

**May:**

* Use registry, services, AD, WMI

**Must:**

* Declare Windows-only intent
* Never assume WSL

---

## 7. Safety & Observability Rules

### 7.1 Safety

* `ReadOnly` must not mutate state
* `Destructive` must:

  * Echo intent
  * Require confirmation
* Cloud scripts must print account / region
* SQL scripts must respect `READ_ONLY`

---

### 7.2 Observability

Scripts must:

* Print start/end markers
* Exit with meaningful codes
* Respect `DRY_RUN`
* Not suppress errors

---

## 8. Final Health Checklist (Mode C)

All must be true:

* [ ] Package ID is stable and unique
* [ ] `package.yaml` validates
* [ ] `.env.example` is complete
* [ ] Primary OS is correct
* [ ] Script tested on primary OS
* [ ] Scope is honest
* [ ] No secrets embedded
* [ ] OS assumptions documented
* [ ] Script runs without launcher
* [ ] Failure modes are understandable

If any item fails → **reject script**.

---

## 9. Failure & Refusal Semantics

* AI systems **must refuse** to generate scripts that violate this playbook
* Reviewers **must block** merge on violations
* Launcher behavior is irrelevant to compliance

---

## 10. What This Playbook Does NOT Do

* Does not execute scripts
* Does not install tools
* Does not manage secrets
* Does not enforce runtime behavior
* Does not replace human judgment

---

## 11. Melissa.ai Integration Contract

Melissa.ai may:

* Invoke this playbook in any mode
* Refuse output when violated
* Require explicit confirmation before proceeding

Melissa.ai **must not** bypass this playbook.

---

## 12. Versioning & Evolution

* This playbook is versioned
* Breaking changes require a new major version
* Scripts should note the playbook version used

