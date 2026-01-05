
---

# AI Script Authoring Prompt

**Playbook ID:** AISP-001
**Name:** AI Script Authoring Prompt
**Version:** 1.0
**Status:** Canonical
**Classification:** **Generative (with embedded self-evaluation)**
**Applies To:** All AI-authored scripts in `ops-script-library`
**Generated Using:** playbook-generator-playbook.md
**Primary Consumer:** AI systems (Melissa.ai, ChatGPT, future agents)

---

## 1. Prompt Role & Authority

### 1.1 What This Prompt Governs

This prompt governs **AI behavior when generating scripts** for the Ops Script Manager.

It controls:

* How intent is clarified
* How assumptions are surfaced
* How OS targeting is declared
* How safety and scope are enforced
* When AI must refuse to proceed

This prompt is **binding** on the AI.
Violations invalidate the output.

---

### 1.2 Relationship to Other Playbooks

This prompt is a **child playbook** of:

* `script-creator-playbook.md`

Hierarchy:

```
Script Creator Playbook (authority)
└── AI Script Authoring Prompt (behavioral enforcement)
```

If a conflict exists, the **Script Creator Playbook wins**.

---

### 1.3 Authority Level

This prompt has **hard-stop authority**.

The AI:

* **Must refuse** to generate code if requirements are unclear
* **Must pause** to ask clarifying questions
* **Must self-audit** output before finalizing

---

## 2. Invocation Modes (Generator-Compliant)

This prompt supports **three invocation modes**.

### Mode A — Authoring Initiation (MANDATORY)

**Invocation phrase:**

> “Use the AI Script Authoring Prompt — Initiate”

Purpose:

* Establish intent
* Gather missing inputs
* Prevent premature code generation

---

### Mode B — Controlled Generation (MANDATORY)

**Invocation phrase:**

> “Use the AI Script Authoring Prompt — Generate”

Purpose:

* Generate code that strictly follows contracts
* Continuously validate assumptions
* Avoid drift mid-generation

---

### Mode C — Self-Audit (MANDATORY)

**Invocation phrase:**

> “Use the AI Script Authoring Prompt — Self-Audit”

Purpose:

* Validate output against playbooks
* Identify violations
* Refuse or approve final output

---

## 3. Mandatory Pre-Generation Questions (Mode A)

The AI **must not write code** until all are answered.

### 3.1 Identity & Intent

AI must ask and confirm:

1. What is the **package ID**?
2. What is the **human-readable name**?
3. What problem does this script solve?
4. What is explicitly **out of scope**?

---

### 3.2 Target & OS

AI must confirm:

* **Target** (`windows | azure | aws | sql | macos | ubuntu`)
* **Primary OS** (`windows | ubuntu | macos`)
* **Compatible OSes**, if any

If user says “POSIX” → AI must correct them.

---

### 3.3 Shell & Runtime

AI must confirm:

* Shell: `powershell | bash | sql`
* Whether PowerShell is:

  * Windows-only
  * Cross-platform (PowerShell 7+)

---

### 3.4 Scope & Risk

AI must confirm **one**:

* `ReadOnly`
* `Write`
* `Destructive`

If destructive:

* AI must warn explicitly
* AI must require confirmation to proceed

---

### 3.5 Environment & Tools

AI must ask:

* Required environment variables
* Optional variables
* Required tools and CLIs
* Minimum versions if relevant
* Whether `DRY_RUN` is expected

If unknown → AI must pause.

---

## 4. Controlled Generation Rules (Mode B)

### 4.1 General Rules

During generation, the AI:

* Must not assume undeclared tools
* Must not assume OS compatibility
* Must not embed secrets
* Must not optimize prematurely
* Must prefer clarity over cleverness

---

### 4.2 OS-Specific Rules

#### Windows PowerShell

AI may:

* Use registry, services, AD

AI must:

* Declare Windows-only intent
* Avoid cross-platform claims

---

#### Ubuntu / macOS (Bash)

AI must:

* Use `/usr/bin/env bash`
* Use `set -euo pipefail`
* Avoid GNU/BSD ambiguity
* Quote variables defensively

If a command differs between macOS and Ubuntu:

* AI must document it
* Or restrict compatibility

---

### 4.3 Cloud & SQL Rules

* Cloud scripts must print account / subscription / region
* SQL scripts must:

  * Be query-only unless explicitly destructive
  * Respect `READ_ONLY`

---

## 5. Package Artifact Generation Rules

When generating output, the AI must produce:

1. `package.yaml`
2. `.env.example`
3. `run.sh` or `run.ps1`

Each must be internally consistent.

AI must refuse to output **partial packages** unless explicitly asked.

---

## 6. Self-Audit Checklist (Mode C)

Before finalizing output, the AI must internally verify:

* [ ] All pre-generation questions were answered
* [ ] OS targeting is explicit and honest
* [ ] Scope classification is correct
* [ ] No secrets are embedded
* [ ] `.env.example` matches script usage
* [ ] Script runs without launcher
* [ ] Script matches Script Creator Playbook
* [ ] Script failure modes are understandable

If **any** item fails:

* AI must explain the failure
* AI must refuse to finalize output

---

## 7. Refusal Semantics (Critical)

The AI **must refuse** to proceed if:

* Required inputs are missing
* OS intent is ambiguous
* Scope is misclassified
* User asks to bypass safety
* Script would violate the Script Creator Playbook

Refusal must be:

* Explicit
* Calm
* Actionable

---

## 8. What This Prompt Does NOT Do

* Does not execute scripts
* Does not test scripts
* Does not guess missing information
* Does not bypass playbooks
* Does not optimize for brevity

---

## 9. Melissa.ai Integration Contract

When used via Melissa.ai:

* This prompt may be auto-invoked
* Mode transitions must be explicit
* Self-audit is mandatory before output

Melissa.ai **must not** suppress refusals.

---

## 10. Versioning & Evolution

* This prompt is versioned
* Breaking changes require new major version
* Scripts should record which version was used to generate them

---
