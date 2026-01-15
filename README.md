Located at: https://raw.githubusercontent.com/lucebryan3000/melissa.ai_artifacts/refs/heads/main/README.md

Rules:
- Only **one agent is authoritative per concern**
- Parallel operation is allowed **only** with clear boundaries
- Conflicts resolve **up the chain**, never by consensus
- Rosie-x is the **only** agent allowed to change code unless Bryan explicitly overrides

---

## 📁 Repository Structure (What Each Area Is)

### `/playbooks` — **PROCESS LAW**
Step-by-step, enforceable procedures.

Playbooks define **how work must be done**.

Key playbooks include:
- `script-creator-playbook.md`
- `script-reviewer-playbook.md`
- `script-lint-playbook.md`
- `harvest-playbook.md`
- `kbgen-queue-playbook.md`
- `kbgen-validate-playbook.md`
- `playbook-generator-playbook.md`
- `lean-skill-playbook.md`

> **Rule:** Playbooks outrank prompts.  
> Deviations must be explicit.

---

### `/prompt` — **INPUT MATERIAL**
Prompt scaffolding and helpers.

Prompts:
- are **not authoritative**
- never override playbooks or agent contracts
- exist to speed up execution, not define process

---

### `/docs` — **SUPPORTING CONTEXT**
Reference material and explanations.

Docs are **supportive**, not authoritative, unless explicitly referenced by a playbook or agent contract.

---

### Root-level artifacts — **CANONICAL SHAPES & CONTRACTS**

These define **what “good” looks like**:

- `prd-playbook.md`
- `frd-playbook.md`
- `techspec-playbook.md`
- `architecture-playbook.md`
- `implementation-playbook.md`
- `migration-playbook.md`
- `api-playbook.md`
- `erd-playbook.md`
- `mermaid-guide.md`
- `adr-template.md`
- `dependency-manager-playbook.md`

These are **structural standards**, not examples.

---

## 🧪 Trust Gate (Build Safety Boundary)

→ **`rubric-judge.md`**

Trust Gate:
- scores artifacts for trustworthiness
- is **non-blocking**
- determines whether an artifact is safe to build from

Tiers:
- **Ready / Solid** → safe build input
- **Needs Work / Not Usable** → publishable, **not** authoritative

This prevents:
- parallel feature creation
- confident wrong builds
- drift from canonical truth

---

## 🔧 KBGEN (Knowledge Base Generation)

KBGEN is the automated pipeline that:
- harvests raw inputs
- generates structured artifacts
- validates and scores them via Trust Gate
- publishes canonical knowledge

KBGEN uses the playbooks and Trust Gate in this repo as **law**, not suggestions.

---

## 🧠 INTasCODE (Governing Philosophy)

**INTasCODE = Intelligence as Code**

Core idea:
> Intelligence lives in **versioned artifacts and pipelines**, not chat history.

This repo exists to make that practical:
- deterministic
- auditable
- reusable
- compounding over time

---

## 🟢 Recommended Session Startup (Copy This Pattern)

When starting a new AI session:

1. Reference this repo’s `README.md`
2. Declare which agent(s) you are invoking:
   - Melissa for decisions
   - HAL9000 for execution
   - Rosie-x for code changes
3. Name the target artifact (PRD, script, spec, etc.)
4. Proceed under the authority model above

---

## 🚫 What This Repo Is Not

- Not a generic AI prompt library
- Not a wiki
- Not chat memory
- Not speculative documentation

If something matters, it becomes an **artifact**.

---

## Final Rule

> **If it isn’t written, validated, and versioned — it isn’t real.**

This repository exists to make that rule usable at speed.
