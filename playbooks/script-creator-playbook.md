# Script Creator Playbook
ID: SCP-001
Version: 1.7
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

- Package ID
  Format: `domain.script-name` (kebab-case, immutable)

- Human Name
  Imperative verb phrase describing exactly what the script does

Example:
```

Package ID: azure.pull-graph-data
Human Name: Pull Microsoft Graph data from Azure AD

````

Options:
1. Single, narrowly scoped script (recommended)
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
- Transitioning from ReadOnly → Write requires a new script
- Destructive scripts require explicit human confirmation

Options:
1. ReadOnly script (recommended)
2. Write script with safeguards
3. Destructive script with mandatory HITL

---

### 4.3 Target & OS Declaration

Choose exactly one target:
- azure
- aws
- m365
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

Options:

1. Single primary OS, optional compatible OS (recommended)
2. Multi-OS primary support
3. OS-agnostic (rare, must justify)

---

### 4.4 Shell & Runtime

Choose exactly one:

* powershell
* bash
* python
* sql

Declare minimum runtime version.

Options:

1. Modern, supported runtime (recommended)
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

Options:

1. Minimal, explicit tooling (recommended)
2. Broad tooling set
3. Tool detection at runtime

---

### 4.6 Environment Contract (MANDATORY, UPDATED)

Every package MUST use `.env`-style files to reduce hard-coding and standardize environment injection.

#### 4.6.1 `.env` Files Are Required

* `.env` files are the authoritative mechanism for documenting and loading environment variables.
* Scripts remain runnable standalone; they MUST be able to load `.env` files without orchestration.
* Orchestration/CI MAY inject environment variables, but scripts MUST behave predictably under that condition (see precedence rules).

#### 4.6.2 Canonical `.env` Naming (OS + Target Layering, No Combos)

Repo-level files (at repo root):

* `.env.master`
* `.env.windows` | `.env.macos` | `.env.ubuntu`
* Optional target layers:

  * `.env.azure` | `.env.aws` | `.env.m365` | `.env.sql`

Package-level files (in the same folder as the package script):

* `<package-id>.env`
* `<package-id>.env.windows` | `<package-id>.env.macos` | `<package-id>.env.ubuntu`
* Optional target layers:

  * `<package-id>.env.azure` | `<package-id>.env.aws` | `<package-id>.env.m365` | `<package-id>.env.sql`

Notes:

* This convention is intentionally explicit to avoid overly-broad `.gitignore` patterns and accidental ignores.

#### 4.6.3 Git Policy: `.example` Shadow Files (MANDATORY)

* Committed files MUST be `*.example`.
* Local secret-bearing files MUST NOT be committed and MUST be git-ignored.
* Runtimes MUST ignore `*.example` at runtime (templates only).

Examples:

* Committed: `.env.master.example`, `.env.windows.example`, `.env.azure.example`, `<package-id>.env.example`, `<package-id>.env.windows.example`, `<package-id>.env.azure.example`
* Local only: `.env.master`, `.env.windows`, `.env.azure`, `<package-id>.env`, `<package-id>.env.windows`, `<package-id>.env.azure`

##### 4.6.3.1 Required `.gitignore` Coverage (MANDATORY)

Repo maintainers MUST ensure `.gitignore` prevents committing secret-bearing `.env` files while allowing `*.example`.

At minimum, the repo MUST ignore:

* `.env.master`
* `.env.windows`
* `.env.macos`
* `.env.ubuntu`
* `.env.azure`
* `.env.aws`
* `.env.m365`
* `.env.sql`

And within package folders (recommended using path-based patterns), ignore:

* `**/*.env`
* `**/*.env.*`

But MUST NOT ignore:

* `**/*.example`

(Exact `.gitignore` implementation is up to repo maintainers, but the behavior above is mandatory.)

#### 4.6.4 Repo Root Discovery Rule (MANDATORY)

All runtimes MUST discover repo root by searching upward from the script/package folder for:

* `master-package.config.psd1`

The first directory containing `master-package.config.psd1` is repo root.

#### 4.6.5 Env Loading Algorithm (MANDATORY, Cross-Runtime)

Each runtime entrypoint MUST implement the same algorithm.

Precedence order (highest → lowest):

1. CLI parameters (if applicable)
2. Process environment variables already set (CI/orchestration/shell)
3. Package OS layer: `<package-id>.env.(windows|macos|ubuntu)`
4. Package target layer: `<package-id>.env.(azure|aws|m365|sql)`
5. Package base: `<package-id>.env`
6. Repo OS layer: `.env.(windows|macos|ubuntu)`
7. Repo target layer: `.env.(azure|aws|m365|sql)`
8. Repo master: `.env.master`

Rules:

* Only load non-`.example` files.
* Missing env files are allowed.
* Env file loading MUST NOT override keys that already exist in the process environment.

  * Env files only fill missing keys.
* If the same key is defined in multiple env files, the higher-precedence source wins.
* Required-variable validation happens after env loading (see §4.6.8).

#### 4.6.6 DRY_RUN Contract (MANDATORY)

Scripts MUST support dry-run as a guardrail where applicable.

* Env var: `DRY_RUN`
* Parameter: `-dry-run` (canonical spelling in docs/examples; runtime-specific parsing is allowed)

Precedence:

* Explicit parameter overrides `DRY_RUN`.
* When dry-run is active, scripts MUST disable external side effects.

##### 4.6.6.1 DRY_RUN Truth Table (MANDATORY)

Case-insensitive:

Truthy:

* `1`, `true`, `yes`, `y`, `on`

Falsey:

* `0`, `false`, `no`, `n`, `off`, empty/unset

If `DRY_RUN` is set to any other value:

* Fail fast with an actionable message.

#### 4.6.7 Dotenv Parsing Contract (MANDATORY, Cross-Runtime)

Parsing MUST be strict and minimal to prevent drift and unsafe behavior.

Allowed:

* `KEY=VALUE`
* Blank lines
* Full-line comments beginning with `#`
* Optional surrounding single or double quotes around VALUE

Not allowed:

* `export KEY=VALUE`
* Variable expansion (`$VAR`, `${VAR}`)
* Multiline values
* Command substitution
* `eval`-style processing
* “Sourcing” the env file as code

Whitespace rules:

* KEY is trimmed.
* For unquoted VALUE: leading/trailing whitespace is trimmed.
* For quoted VALUE: outer quotes are removed; inner whitespace is preserved exactly.

#### 4.6.8 Required vs Optional Variables (MANDATORY)

Every script MUST define which environment variables it requires and validate them.

Rules:

* Runtime enforcement source of truth: the script defines a `RequiredEnvVars` list (runtime-native construct) and validates after env loading.
* Documentation: frontmatter MUST list required and optional env vars with brief purpose statements.
* `.env.*.example` files mirror the documented variables as templates (derivative; never runtime-loaded).

---

### 4.7 Repo Configuration & Bootstrap Contract (MANDATORY, NEW)

This contract is PowerShell-scoped for config/bootstrap, but repo-root discovery is global (see §4.6.4).

#### 4.7.1 Authoritative Defaults (Repo-Wide)

* Root config is `master-package.config.psd1` (authoritative defaults).

#### 4.7.2 PowerShell Bootstrap (PowerShell Runtime Only)

If `runtime=powershell`, scripts/tools MUST:

* Locate repo root by searching upward for:

  * `master-package.config.psd1`
  * `ops.bootstrap.psm1`
* Import `ops.bootstrap.psm1`
* Call `Import-OpsPackageConfig`
* MUST NOT manually merge configs

#### 4.7.3 Config Override Model (Rare)

Default behavior:

* Use only `master-package.config.psd1`

Optional overlay (rare):

* If `package.config.psd1` exists in the same folder as the script/package, it overlays master.

---

### 4.8 Package Layout Mapping (MANDATORY, NEW)

Strict package mapping is required for all packages.

Rules:

* Package folder name MUST equal Package ID
* Primary entry script name MUST equal Package ID (with runtime extension)

Required paths:

* Package folder: `/<target>/<package-id>/`
* Primary script: `/<target>/<package-id>/<package-id>.<ext>`

Reserved subfolders (when applicable):

* `artifacts/` → generated outputs (reports, json, csv, diagrams) and governance outputs produced by governance tooling
* `examples/` → runnable examples (local/ci)

---

### Proceed to Next Play

➡️ Prompt: Run Script Creation (Mode B)
➡️ Next recommended play after completion: Script Lint Playbook

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
* Parameters (including `-dry-run` where applicable)
* Environment variables:

  * Required vars list
  * Optional vars list
  * DRY_RUN behavior
  * `.env` contract summary (what files it loads, and precedence)
* Usage examples (including `-dry-run` examples where applicable)
* `--help` / `-Help` behavior

---

### 5.1.1 Env Loader Requirements (All Runtimes) (MANDATORY)

Each runtime MUST implement env loading per Mode A §4.6.

Security requirements:

* Implement parsing as data parsing, never as code execution.
* Bash MUST NOT `source` env files and MUST NOT use `eval`.
* PowerShell MUST NOT dot-source env files and MUST NOT invoke expressions.
* Python MUST parse `.env` files without executing code.

Behavioral requirements:

* Must discover repo root by searching for `master-package.config.psd1` (see §4.6.4).
* Must ignore `*.example` at runtime.
* Must not override pre-set process environment variables.
* Must validate required env vars after loading (see §4.6.8).

---

### 5.1.2 PowerShell Script Creation Standard (Consistency + Repo Decisions)

This subsection defines the mandatory, repeatable structure for creating PowerShell scripts in `ops-script-library`.

#### A) Mandatory Script Skeleton (PowerShell)

Every PowerShell script MUST follow this internal structure:

1. Frontmatter + Comment-Based Help

* `.SYNOPSIS`, `.DESCRIPTION`, `.PARAMETER`, `.EXAMPLE`, `.NOTES`
* MUST match Mode A declarations

2. Repo Root Discovery (MANDATORY)

* Determine package folder (script directory)
* Discover repo root by searching upward for `master-package.config.psd1`

3. `.env` Loading (MANDATORY)

* Implement Mode A §4.6 env loading algorithm and parsing contract
* Ignore `*.example` at runtime
* Do NOT override existing process env variables
* Validate required env vars after loading

4. Bootstrap + Config (PowerShell Runtime)

* Import `ops.bootstrap.psm1`
* Call `Import-OpsPackageConfig`
* MUST NOT manually merge configs
* Optional overlay (rare): `package.config.psd1` in same folder overlays master

5. Requires / Version / Dependencies

* Declare minimum PowerShell version (and required modules if applicable)
* Fail fast if prerequisites are not met

6. `[CmdletBinding()]` + Parameter Block

* Use advanced function style even in scripts
* Implement a `-DryRun` switch parameter and document spelling as `-dry-run` in examples

  * Parameter overrides `DRY_RUN`
* Parameter attributes:

  * types
  * validation (`ValidateSet/Pattern/Range/NotNullOrEmpty`)
  * parameter sets if needed

7. Begin/Process/End Pattern (when pipeline input is supported)

* `begin {}`: env validation + dependency checks + setup
* `process {}`: core work per input item
* `end {}`: final output/summary, cleanup

8. Output Contract (Objects First)

* Return structured objects or JSON
* Do not call `Format-*` in core logic
* Avoid `Write-Host` except explicitly interactive UX
* Use `Write-Verbose`, `Write-Information`, `Write-Warning`, `Write-Error`

9. Mutations Must Use `ShouldProcess`

* If scope is Write/Destructive:

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
* When dry-run is active (env or param), external side effects MUST be disabled

12. Logging / Auditability

* Emit structured logs (recommended key/value or JSON)
* Include at minimum:

  * script/package ID
  * timestamp
  * correlation/run ID
  * target identifier(s)
  * scope
  * result status per item

#### B) PowerShell-Specific Rules (Non-Negotiable)

* Use approved verbs (`Get-Verb`)
* Prefer modules for reusable logic; scripts should orchestrate
* Avoid global state; no reliance on current directory unless explicitly declared
* Secrets must come from env vars/managed identity; no interactive credential prompts
* Do not use `Write-Host` for operational output (interactive UX only)
* Return objects; formatting is caller responsibility
* If destructive: require HITL confirmation (as defined in §5.4)

---

### 5.1.3 Target Templates (Placeholders for Other Targets)

All target templates MUST inherit global rules:

* Mode A §4.6 env contract (naming, root discovery, precedence, parsing, required-var validation, ignore `.example`)
* Mode B §5.2 determinism/output rules
* Mode B §5.3 error handling rules
* Mode B §5.4 HITL rules
* Mode B §5.5 scope guardrails

These are placeholders to be expanded over time.

#### 5.1.3.1 Windows Script Template (Placeholder)

Applies to: Windows-targeted automation not implemented in PowerShell (e.g., native tooling, vendor CLIs)

Minimum requirements:

* Standardized frontmatter
* Implements env loading per §4.6
* Deterministic structured output
* Meaningful exit codes
* DRY_RUN where applicable

#### 5.1.3.2 Ubuntu Script Template (Placeholder)

Applies to: bash or Linux-native operational scripting

Minimum requirements:

* Standardized frontmatter
* Implements env loading per §4.6 (MUST NOT `source` or `eval`)
* Deterministic structured output (JSON preferred)
* Meaningful exit codes
* DRY_RUN where applicable

#### 5.1.3.3 AWS Script Template (Placeholder)

Applies to: scripts that interact with AWS (CLI/SDK/Terraform wrappers, etc.)

Minimum requirements:

* Standardized frontmatter
* Implements env loading per §4.6
* Account/region guardrails
* Deterministic structured output
* DRY_RUN where applicable

#### 5.1.3.4 Azure Script Template (Placeholder)

Applies to: scripts that interact with Azure (Az CLI/Az PowerShell/Graph/etc.)

Minimum requirements:

* Standardized frontmatter
* Implements env loading per §4.6
* Tenant/subscription guardrails
* Deterministic structured output
* DRY_RUN where applicable

#### 5.1.3.5 macOS Script Template (Placeholder)

Applies to: zsh/bash scripts, vendor tools, MDM-related automation, etc.

Minimum requirements:

* Standardized frontmatter
* Implements env loading per §4.6 (MUST NOT `source` or `eval`)
* Deterministic structured output
* Meaningful exit codes
* DRY_RUN where applicable

#### 5.1.3.6 SQL Script Template (Placeholder)

Applies to: SQL scripts used for reporting, validation, migrations, and operational maintenance

Minimum requirements:

* Standardized frontmatter
* Environment contract documented (how connection info is provided via env)
* Transaction safety rules
* ReadOnly vs Write vs Destructive rules
* Deterministic output schema

#### 5.1.3.7 Python Script Template (Placeholder)

Applies to: Python operational scripts

Minimum requirements:

* Standardized frontmatter
* Implements env loading per §4.6 (parse-only, no code execution)
* Deterministic structured output (JSON preferred)
* Meaningful exit codes
* DRY_RUN where applicable

---

### 5.2 Determinism & Output Rules

Scripts MUST:

* Avoid interactive prompts by default
* Return structured data (objects, JSON)
* Avoid formatted output
* Emit timestamps explicitly
* Clearly declare side effects

---

### 5.2.1 Orchestration Compatibility (MANDATORY)

Orchestration modes are:

* `plan` (orchestrator-only)
* `dry-run` (orchestrator invokes scripts with `-dry-run` and/or `DRY_RUN=...`)
* `run`

Rules:

* Scripts MUST remain runnable standalone.
* Scripts MUST support script-level dry-run control (`-dry-run` and/or `DRY_RUN`) where applicable.
* Scripts are NOT required to implement `-mode plan|dry-run|run`. `plan` remains orchestrator-only.

---

### 5.2.2 Artifacts & Governance Philosophy (GLOBAL RULE)

Operational scripts MUST follow the repo’s artifacts philosophy:

* No placeholder governance artifacts.
* Governance outputs are generated only when governance steps run.
* Operational scripts MUST NOT create governance artifacts.

---

### 5.2.3 Governance Artifact Contract (Combined) (MANDATORY)

Governance outputs are consolidated into one file to simplify manageability.

#### Output File (Generated Only When Governance Runs)

* `<package>/artifacts/governance.yaml`

#### Governance Config (Master + Override)

* Master config (committed): `<repo-root>/master-governance.config.yaml`
* Package override config (committed, rare): `<package>/<package-id>.governance.config.yaml`

Rules:

* Overrides are intended to be rare and must be justified.
* Tools SHOULD load master config, then apply package override if present.

#### governance.yaml Schema (v1)

Top-level keys:

* `schema_version`
* `package_id`
* `generated_at`

Sections:

* `lint: { ... }`
* `review: { ... }`
* `policy: { ... }`

#### Write/Merge Rules (MANDATORY)

* Each producer step overwrites ONLY its own top-level section:

  * Lint tooling writes/updates only `lint`
  * Review process writes/updates only `review`
  * Policy tooling writes/updates only `policy`
* A step MUST NOT edit or delete other sections.

#### Legacy Compatibility (Transition)

* Legacy split files (`lint.yaml`, `review.yaml`, `policy.yaml`) are considered deprecated.
* New governance runs MUST NOT create split files.
* Tooling MAY optionally read legacy inputs during transition, but authoritative output is `governance.yaml`.

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
  * Support dry-run where feasible
  * Escalate to HITL if risk is high
* Any Write action MUST be obvious from the script name and frontmatter

---

### Proceed to Next Play

➡️ Prompt: Run Script Lint Playbook
➡️ Next recommended play after lint: Script Reviewer Playbook

---

## 6. Mode C — Script Lint (REQUIRED)

Linting MUST:

* Be explicitly executed
* Follow Script Lint Playbook
* Produce PASS / WARN / FAIL
* Record findings and remediation

Manual best practices do NOT count as linting.

Options:

1. Automated lint with recorded output (recommended)
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

Options:

1. Independent reviewer with checklist (recommended)
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

1. Run Script Lint Playbook (recommended first)
2. Run Script Reviewer Playbook
3. Proceed to Packaging or Orchestration

---

End of Playbook
