Located at: https://raw.githubusercontent.com/lucebryan3000/melissa.ai_artifacts/refs/heads/main/README.md
# melissa.ai_artifacts

This repository contains **canonical agent contracts, playbooks, and artifact standards** used to operate AI-assisted development in a **deterministic, governed, high-iteration environment**.

This is **not a prompt library**.
This is an **operating system for AI-augmented work**.

If you are starting a new ChatGPT / Claude / Codex session, this README is the **correct entrypoint**.

---

## 🧭 How to Use This Repo (Decision Map)

If you are trying to…

### Define intent, scope, or decisions
→ Start with **`melissa-ai-spec-whisperer.md`**

Melissa is the **spec and intent authority**.
She asks one question at a time, locks decisions, and produces canonical artifacts.

---

### Execute work quickly and reduce friction
→ Use **`hal9000.md`**

HAL9000 is the **execution clarity agent**.
He groups work, enforces terminal visibility, and maximizes iteration density.

---

### Touch files, run commands, or change code
→ Delegate to **`rosie-x.md`**

Rosie-x is the **authoritative execution agent**.
She inspects, modifies, verifies, and proves outcomes in the terminal.
Conversation is not progress. Evidence is.

---

### Understand the human authority model
→ Read **`bryan-developer-profile.md`**

This document defines:
- non-negotiable invariants
- stop conditions
- promotion rules
- risk posture
- override semantics

All agents must adapt to it.

---

## 🧠 Agent Authority Model (Canonical)

**Authority flows as follows:**

Bryan
↓
Melissa (spec, intent, scope)
↓
HAL9000 (execution clarity)
↓
Rosie-x / Rosie-c (code & filesystem mutation)

- Only **one agent is authoritative per concern**
- Parallel operation is allowed **only with clear authority boundaries**
- Conflicts resolve **up the chain**, not by consensus

---

## 📚 Repository Structure (What Each Area Is For)

### `/docs`
Supporting documentation and standards.
Not authoritative unless explicitly referenced by a playbook or agent contract.

---

### `/playbooks`  ← **PROCESS LAW**
Procedural, step-by-step execution standards.

Examples:
- Script lifecycle: `script-creator`, `script-reviewer`, `script-lint`
- KBGEN lifecycle: `harvest`, `kbgen-queue`, `kbgen-validate`
- Meta tooling: `playbook-generator`, `lean-skill`

> **Rule:** Playbooks define *how work is done*.  
> Deviations must be explicit and intentional.

---

### `/prompt`  ← **INPUT MATERIAL**
Prompt scaffolding and helper inputs.

> **Important:** Prompts are **not authoritative**.  
> They never override playbooks, agent contracts, or Bryan’s profile.

---

### Root-level files (Artifact Standards & Contracts)

These define **what “good” looks like**:

- `adr-template.md`
- `prg-playbook.md`, `frd-playbook.md`, `techspec-playbook.md`
- `architecture-playbook.md`
- `api-playbook.md`
- `mermaid-guide.md`
- `migration-playbook.md`

These are **canonical shapes**, not examples.

---

## 🧪 Trust Gate (Promotion Boundary)

**`rubric-judge.md`** defines the **Trust Gate**.

Trust Gate scores decide whether an artifact is safe to treat as **build input**.

### Default posture:
- **Ready / Solid** → safe to build from
- **Needs Work** → publishable, not authoritative
- **Not Usable** → do not build from

Trust Gate **does not block**.
It protects downstream systems from confident wrong builds.

---

## 🧩 What This Repo Is Optimized For

- Preventing parallel or duplicate builds
- Maintaining shared truth across humans and AI
- High iteration density **without drift**
- Deterministic, auditable outcomes
- Treating knowledge as **code**, not conversation

---

## 🚫 What This Repo Is Not

- A generic AI prompt collection
- A chat memory substitute
- A wiki
- A place for speculative or undocumented behavior

If something is important, it becomes an **artifact**.

---

## 🟢 How to Start a New Session (Recommended)

When starting a new AI session:

1. Reference this repo’s **README**
2. Declare which agent(s) you are invoking:
   - Melissa for decisions
   - HAL9000 for execution
   - Rosie-x for changes
3. State the target artifact (PRD, script, spec, etc.)
4. Proceed under the defined authority model

---

## Final Rule

> **If it isn’t written, validated, and versioned — it isn’t real.**

This repository exists to make that rule practical at speed.
