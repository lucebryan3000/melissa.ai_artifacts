# Script Creator Playbook  
ID: SCP-001  
Version: 1.5  
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
5. Declare scope honestly (ReadOnly vs Write vs Destructive)  
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
   Reason: Easier to review, lint, automate, and reason about  
2. Multi-purpose script  
3. Script that delegates to other scripts  

---

### 4.2 Script Type & Scope Declaration (MANDATORY)

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
   Reason: Safest default, easiest to automate  
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
* DRY_RUN support expectations

Rules:

* Secrets via environment variables or managed identity only
* No prompting for secrets
* DRY_RUN disables all external side effects (when applicable)

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

### 5.1 Standardized Frontmatter (MANDATORY)

Every script MUST include standardized frontmatter, regardless of type.

Frontmatter MUST declare:

* Synopsis
* Description
* Script type
* Scope (ReadOnly / Write / Destructive)
* Target platform
* OS support
* Runtime requirements
* Tooling requirements
* Parameters
* Environment variables
* Usage examples
* `--help` / `-Help` behavior

This is REQUIRED for **all scripts**, not optional.

---

### 5.1.1 PowerShell Script Creation Standard (Consistency + Repo Decisions)

This subsection defines the mandatory, repeatable structure for creating PowerShell scripts in `ops-script-library`. Every PowerShell script MUST conform so that humans and AI systems can review, run, lint, automate, orchestrate, and govern consistently.

#### A) Mandatory Script Skeleton (PowerShell)

Every PowerShell script MUST follow this internal structure:

1. Frontmatter + Comment-Based Help

* `.SYNOPSIS`, `.DESCRIPTION`, `.PARAMETER`, `.EXAMPLE`, `.NOTES`
* MUST match Mode A declarations (type, scope, target, OS, runtime, tooling, env vars)

2. Repo Root Discovery + Bootstrap Import (MANDATORY)

* Scripts/tools MUST locate repo root by searching upward for:

  * `master-package.config.psd1`
  * `ops.bootstrap.psm1`
* Scripts/tools MUST import `ops.bootstrap.psm1` and call `Import-OpsPackageConfig`
* Scripts/tools MUST NOT manually merge configs

3. Config Resolution Order (MANDATORY)

* Default behavior: use only `master-package.config.psd1` (authoritative defaults)
* Optional overlay (rare): if `package.config.psd1` exists **in the same folder as the script/package**, overlay it on master
* Overlay is intended to be rare; do not create `package.config.psd1` unless justified during Mode A

4. Requires / Version / Dependencies

* Declare minimum PowerShell version (and required modules if applicable)
* Fail fast if prerequisites are not met

5. `[CmdletBinding()]` + Parameter Block

* Use advanced function style even in scripts
* Parameter attributes:

  * types
  * validation (`ValidateSet/Pattern/Range/NotNullOrEmpty`)
  * parameter sets if needed

6. Begin/Process/End Pattern (when pipeline input is supported)

* `begin {}`: environment validation + dependency checks + setup
* `process {}`: core work per input item
* `end {}`: final output/summary, cleanup

7. Environment Validation and Context Lock

* Validate:

  * required env vars
  * connectivity/auth context
  * permissions and target reachability
* If validation fails: terminate with actionable error

8. Output Contract (Objects First)

* Scripts MUST return structured objects or JSON
* Do not call `Format-*` in core logic
* Avoid `Write-Host` except explicitly interactive UX
* Use `Write-Verbose`, `Write-Information`, `Write-Warning`, `Write-Error`

9. Mutations Must Use `ShouldProcess`

* If scope is Write/Destructive, script MUST:

  * implement `SupportsShouldProcess`
  * honor `-WhatIf` and `-Confirm`
  * make side effects explicit

10. Error Handling Contract

* Use `try/catch/finally`
* Use `-ErrorAction Stop` when the intention is to catch
* Never swallow exceptions; handle or rethrow with context
* Exit codes must be meaningful for automation (non-zero on fatal)

11. Determinism / Idempotence

* No prompts unless `-Interactive` (or `-Confirm` for destructive actions)
* Re-runs should not drift; check state before changing state
* DRY_RUN is supported as a guardrail where applicable; DRY_RUN MUST disable external side effects

12. Logging / Auditability

* Emit structured logs (recommended key/value or JSON)
* Include at minimum:

  * script ID
  * timestamp
  * correlation/run ID
  * target identifier(s)
  * scope (ReadOnly/Write/Destructive)
  * result status per item

#### B) PowerShell-Specific Rules (Non-Negotiable)

* Use approved verbs (`Get-Verb`)
* Prefer modules for reusable logic; scripts should orchestrate
* Avoid global state; no reliance on current directory unless explicitly declared
* Secrets must come from env vars/managed identity; no interactive credential prompts
* Do not use `Write-Host` for operational output (interactive UX only)
* Return objects; formatting is caller responsibility
* If destructive: require HITL confirmation (as defined in 5.4)

#### C) Artifacts & Governance (Repo Decision Alignment)

Operational scripts MUST follow the repo’s artifacts philosophy:

* No placeholder governance artifacts.
* `lint.yaml`, `review.yaml`, `policy.yaml` are generated only when their governance steps run.
* Operational scripts do not create governance artifacts.
* Script-produced operational outputs MAY be written into `artifacts/` (reports, json, csv, diagrams) only when the script’s purpose explicitly calls for it and the output contract documents it.

Governance tooling behavior (for awareness):

* `Invoke-ScriptLint.ps1` writes `lint.yaml` (v1.1) to the package’s `artifacts/`.
* `Evaluate-Policy.ps1` reads `lint.yaml` + `review.yaml` and writes `policy.yaml`.
* `review.yaml` is produced by the review process (human/AI), not preseeded.

---

#### PowerShell Script Creation Checklist (SCP-001 / Mode B)

Identity & Declaration

* [ ] Package ID and Human Name present and accurate
* [ ] Script type declared (Operational/Validation/Reporting/Orchestration)
* [ ] Scope declared and accurate (ReadOnly/Write/Destructive)
* [ ] Target declared (azure/aws/windows/macos/ubuntu/sql)
* [ ] OS support declared (primary + compatible list)
* [ ] Minimum PowerShell version declared

Repo Bootstrap & Config

* [ ] Repo root discovery uses upward search for `master-package.config.psd1` + `ops.bootstrap.psm1`
* [ ] Script imports `ops.bootstrap.psm1` and calls `Import-OpsPackageConfig`
* [ ] Script does not manually merge configs
* [ ] Optional `package.config.psd1` overlay is supported (only if present in same folder) and justified

Interface & Help

* [ ] Comment-based help includes Synopsis, Description, Parameters, Examples
* [ ] `-Help` / `--help` behavior documented (or implemented)

Parameters

* [ ] `[CmdletBinding()]` used
* [ ] Parameters typed and validated
* [ ] Parameter sets used if ambiguity exists
* [ ] Defaults are safe for automation (non-interactive)

Environment & Dependencies

* [ ] Required tools/modules listed with minimum versions (no floating versions)
* [ ] Required env vars declared; optional env vars declared
* [ ] DRY_RUN behavior implemented where applicable (disables side effects)
* [ ] Preflight validation happens before any external action

Execution Safety

* [ ] No prompts unless `-Interactive` (and only in TTY)
* [ ] Write/Destructive actions use `ShouldProcess` (`-WhatIf` supported)
* [ ] Side effects explicit in name + help + output/logs
* [ ] Idempotence considered (safe to re-run)

Artifacts & Governance

* [ ] Script does not create placeholder governance artifacts (`lint.yaml`, `review.yaml`, `policy.yaml`)
* [ ] If script writes operational outputs to `artifacts/`, the output contract documents what/where/format

Output & Logging

* [ ] Outputs structured objects or JSON
* [ ] No `Format-*` in core logic
* [ ] Logging uses standard streams (`Verbose/Information/Warning/Error`)
* [ ] Run ID/correlation ID present in logs/output

Errors & Exit Codes

* [ ] Inputs validated early
* [ ] `try/catch/finally` used where external calls occur
* [ ] Errors include actionable context
* [ ] Non-zero exit on fatal errors (automation-safe)

Ready for Lint/Review

* [ ] PSScriptAnalyzer clean or issues documented with justification
* [ ] Pester tests exist when logic is non-trivial (or explicitly waived with rationale)
* [ ] Script is small/single-purpose; reusable parts moved to module functions

---

### 5.1.2 Script Type Templates (Placeholders for Other Targets)

This section reserves standardized creation templates for other targets and runtimes. These templates will be filled in over time. Even though multiple script types (Operational/Validation/Reporting/Orchestration) may apply to any target, each target template defines the consistent baseline structure, safety rules, and output/error contracts for that runtime.

#### 5.1.2.1 Windows Script Template (Placeholder)

Applies to: Windows-targeted automation not implemented in PowerShell (e.g., native tooling, vendor CLIs)

Minimum requirements (to be defined):

* Frontmatter standard
* Non-interactive defaults
* Deterministic output contract
* Error/exit code contract
* Logging/audit contract
* Dependency declaration and version pinning
* DRY_RUN / simulation strategy

Checklist (placeholder)

* [ ] Standardized frontmatter present and matches Mode A declarations
* [ ] Dependencies + versions declared; preflight validates them
* [ ] Non-interactive by default; interactive gated behind explicit flag
* [ ] Deterministic structured output produced
* [ ] Exit codes meaningful; failures are actionable
* [ ] Side effects explicit; DRY_RUN/sim mode supported

---

#### 5.1.2.2 Ubuntu Script Template (Placeholder)

Applies to: bash or Linux-native operational scripting

Minimum requirements (to be defined):

* Frontmatter standard (header block)
* Shell safety settings (`set -euo pipefail` pattern or equivalent)
* Deterministic output contract (JSON preferred)
* Dependency/version checks (command presence + version gating)
* DRY_RUN/simulation strategy
* Logging/audit contract
* Exit code contract

Checklist (placeholder)

* [ ] Standardized frontmatter present and matches Mode A declarations
* [ ] Safe shell options enabled (or explicitly justified if not)
* [ ] Dependencies validated before side effects
* [ ] Non-interactive by default; prompts gated behind explicit flag
* [ ] Structured output produced (JSON preferred)
* [ ] Exit codes meaningful; failures are actionable
* [ ] DRY_RUN/sim mode supported

---

#### 5.1.2.3 AWS Script Template (Placeholder)

Applies to: scripts that interact with AWS (CLI/SDK/Terraform wrappers, etc.)

Minimum requirements (to be defined):

* Frontmatter standard
* Auth context validation (profile/role/identity)
* Region/account guardrails
* Deterministic output contract
* DRY_RUN strategy (where feasible)
* Change safety (`plan` vs `apply` semantics where applicable)
* Logging/audit contract (include account, region, resource identifiers)

Checklist (placeholder)

* [ ] Frontmatter includes account/region expectations and scope
* [ ] Auth validated (whoami/identity) before any actions
* [ ] Account/region guardrails enforced
* [ ] Write actions gated (plan/preview + confirmation where high risk)
* [ ] Structured output produced
* [ ] Errors actionable; exit codes meaningful
* [ ] DRY_RUN/sim mode supported

---

#### 5.1.2.4 Azure Script Template (Placeholder)

Applies to: scripts that interact with Azure (Az CLI/Az PowerShell/Graph/etc.)

Minimum requirements (to be defined):

* Frontmatter standard
* Tenant/subscription guardrails
* Identity validation (signed-in principal, managed identity, service principal)
* Deterministic output contract
* DRY_RUN strategy (where feasible)
* Change safety (preview/confirm)
* Logging/audit contract (tenant, subscription, resource IDs)

Checklist (placeholder)

* [ ] Frontmatter includes tenant/subscription expectations and scope
* [ ] Identity validated before any actions
* [ ] Tenant/subscription guardrails enforced
* [ ] Write actions gated (preview/confirm for risk)
* [ ] Structured output produced
* [ ] Errors actionable; exit codes meaningful
* [ ] DRY_RUN/sim mode supported

---

#### 5.1.2.5 macOS Script Template (Placeholder)

Applies to: zsh/bash scripts, vendor tools, MDM-related automation, etc.

Minimum requirements (to be defined):

* Frontmatter standard
* Shell safety settings
* Dependency/version checks
* Deterministic output contract
* DRY_RUN strategy
* Logging/audit contract
* Exit code contract

Checklist (placeholder)

* [ ] Standardized frontmatter present and matches Mode A declarations
* [ ] Safe shell options enabled (or explicitly justified if not)
* [ ] Dependencies validated before side effects
* [ ] Non-interactive by default; prompts gated behind explicit flag
* [ ] Structured output produced (JSON preferred)
* [ ] Exit codes meaningful; failures are actionable
* [ ] DRY_RUN/sim mode supported

---

#### 5.1.2.6 SQL Script Template (Placeholder)

Applies to: SQL scripts used for reporting, validation, migrations, and operational maintenance

Minimum requirements (to be defined):

* Frontmatter metadata (target DB, environment, scope)
* Transaction safety rules
* ReadOnly vs Write vs Destructive rules
* Deterministic output contract (schema + columns defined)
* Logging/audit strategy (execution tracking)
* Error handling / rollback expectations

Checklist (placeholder)

* [ ] Frontmatter includes target DB/environment and scope classification
* [ ] Transactions used appropriately; rollback strategy defined
* [ ] ReadOnly/Write/Destructive behavior explicit
* [ ] Output schema defined and consistent
* [ ] Execution is auditable (who/when/where)
* [ ] Failures handled predictably

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

### 5.5 ReadOnly vs Write Guardrails (MANDATORY)

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
   Why: catches defects before human review

2. **Run Script Reviewer Playbook**
   Why: formal approval and risk acknowledgment

3. **Proceed to Packaging or Orchestration**
   Why: only approved scripts should be automated

---

End of Playbook
