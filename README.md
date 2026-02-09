# melissa.ai_artifacts

> **Canonical source for AI-assisted development workflows and artifact generation**

Located at: https://raw.githubusercontent.com/lucebryan3000/melissa.ai_artifacts/refs/heads/main/README.md

---

## 🎯 What This Repository Contains

This repository provides **referenceable, authoritative artifacts** for AI-assisted development:

- **Playbooks** - Enforceable process definitions
- **Templates** - Specification and documentation structures
- **Agent Profiles** - AI agent contracts and personas
- **Standards** - Development guidelines and quality rubrics
- **Guides** - Reference material for tools and patterns

### ✅ How to Reference Artifacts

All files in this repository are designed to be:
1. **Directly referenced** in AI prompts
2. **Copied** as templates for new projects
3. **Extended** with project-specific adaptations
4. **Cited** as authoritative sources

Use markdown links to reference specific files in AI conversations:
- `[prd-playbook.md](https://raw.githubusercontent.com/lucebryan3000/melissa.ai_artifacts/refs/heads/main/prd-playbook.md)`
- Or use relative links within the repo: `[prd-playbook.md](prd-playbook.md)`

---

## 🏛️ Authority Model

Rules:
- Only **one agent is authoritative per concern**
- Parallel operation is allowed **only** with clear boundaries
- Conflicts resolve **up the chain**, never by consensus
- [rosie-x.md](rosie-x.md) is the **only** agent allowed to change code unless Bryan explicitly overrides

---

## 📁 Repository Structure (What Each Area Is)

### `/playbooks/` — **PROCESS LAW**

Step-by-step, enforceable procedures.

Playbooks define **how work must be done**.

**Key playbooks**:
- [script-creator-playbook.md](playbooks/script-creator-playbook.md) - Script creation standards
- [script-reviewer-playbook.md](playbooks/script-reviewer-playbook.md) - Script review process
- [script-lint-playbook.md](playbooks/script-lint-playbook.md) - Linting procedures
- [harvest-playbook.md](playbooks/harvest-playbook.md) - Knowledge harvesting
- [kbgen-queue-playbook.md](playbooks/kbgen-queue-playbook.md) - KB generation queue
- [kbgen-validate-playbook.md](playbooks/kbgen-validate-playbook.md) - KB validation
- [playbook-generator-playbook.md](playbooks/playbook-generator-playbook.md) - Meta-playbook for creating playbooks
- [lean-skill-playbook.md](playbooks/lean-skill-playbook.md) - Lean skill development
- [github-project-setup.md](playbooks/github-project-setup.md) - GitHub project initialization

> **Rule:** Playbooks outrank prompts.
> Deviations must be explicit.

---

### `/prompt/` — **INPUT MATERIAL**

Prompt scaffolding and helpers.

Prompts:
- are **not authoritative**
- never override playbooks or agent contracts
- exist to speed up execution, not define process

---

### `/docs/` — **SUPPORTING CONTEXT**

Reference material and explanations.

Docs are **supportive**, not authoritative, unless explicitly referenced by a playbook or agent contract.

**Key documents**:
- [docs/document-types/](docs/document-types/) - Document type definitions and schemas
- [docs/standards/](docs/standards/) - Development standards

---

### Root-level artifacts — **CANONICAL SHAPES & CONTRACTS**

These define **what "good" looks like**:

**Specification Playbooks**:
- [prd-playbook.md](prd-playbook.md) - Product Requirements Document structure
- [frd-playbook.md](frd-playbook.md) - Functional Requirements Document structure
- [techspec-playbook.md](techspec-playbook.md) - Technical Specification structure
- [architecture-playbook.md](architecture-playbook.md) - Architecture documentation
- [implementation-playbook.md](implementation-playbook.md) - Implementation planning
- [migration-playbook.md](migration-playbook.md) - Data migration procedures
- [api-playbook.md](api-playbook.md) - API design and documentation
- [erd-playbook.md](erd-playbook.md) - Entity Relationship Diagram creation

**Templates & Standards**:
- [adr-template.md](adr-template.md) - Architecture Decision Record template
- [dependency-manager-playbook.md](dependency-manager-playbook.md) - Dependency management
- [mermaid-guide.md](mermaid-guide.md) - Mermaid diagram syntax reference
- [prompt-playbook.md](prompt-playbook.md) - Prompt engineering standards
- [claude-slash-command.md](claude-slash-command.md) - Slash command creation guide

**Agent Profiles**:
- [melissa-ai-spec-whisperer.md](melissa-ai-spec-whisperer.md) - Specification writing assistant
- [hal9000.md](hal9000.md) - Execution orchestrator
- [rosie-x.md](rosie-x.md) - Code operator (sole code authority)
- [bryan-developer-profile.md](bryan-developer-profile.md) - Developer standards profile

**Quality & Evaluation**:
- [rubric-judge.md](rubric-judge.md) - Quality evaluation rubric (Trust Gate)

These are **structural standards**, not examples.

---

## 🧪 Trust Gate (Build Safety Boundary)

→ [rubric-judge.md](rubric-judge.md)

Trust Gate:
- scores artifacts for trustworthiness
- is **non-blocking**
- determines whether an artifact is safe to build from

**Tiers**:
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
- **deterministic** - Same input → same output
- **auditable** - All changes versioned and traceable
- **reusable** - Templates and playbooks work across projects
- **compounding** - Knowledge accumulates over time

---

## 🟢 Recommended Session Startup (Copy This Pattern)

When starting a new AI session:

1. **Reference this repo**: `[README.md](https://raw.githubusercontent.com/lucebryan3000/melissa.ai_artifacts/refs/heads/main/README.md)`
2. **Declare agent(s)**:
   - [melissa-ai-spec-whisperer.md](melissa-ai-spec-whisperer.md) for decisions
   - [hal9000.md](hal9000.md) for execution
   - [rosie-x.md](rosie-x.md) for code changes
3. **Name target artifact** (PRD, script, spec, etc.)
4. **Reference relevant playbook** from the list above
5. Proceed under the authority model

---

## 📋 Quick Reference - All Artifacts

### Playbooks (Process)
- [playbooks/ai-script-authoring-prompt.md](playbooks/ai-script-authoring-prompt.md)
- [playbooks/github-project-setup.md](playbooks/github-project-setup.md)
- [playbooks/harvest-playbook.md](playbooks/harvest-playbook.md)
- [playbooks/kbgen-queue-playbook.md](playbooks/kbgen-queue-playbook.md)
- [playbooks/kbgen-validate-playbook.md](playbooks/kbgen-validate-playbook.md)
- [playbooks/lean-skill-playbook.md](playbooks/lean-skill-playbook.md)
- [playbooks/playbook-generator-playbook.md](playbooks/playbook-generator-playbook.md)
- [playbooks/script-creator-playbook.md](playbooks/script-creator-playbook.md)
- [playbooks/script-lint-playbook.md](playbooks/script-lint-playbook.md)
- [playbooks/script-reviewer-playbook.md](playbooks/script-reviewer-playbook.md)

### Specification Playbooks (Structure)
- [prd-playbook.md](prd-playbook.md) - Product Requirements
- [frd-playbook.md](frd-playbook.md) - Functional Requirements
- [techspec-playbook.md](techspec-playbook.md) - Technical Specifications
- [architecture-playbook.md](architecture-playbook.md) - Architecture
- [implementation-playbook.md](implementation-playbook.md) - Implementation
- [migration-playbook.md](migration-playbook.md) - Migrations
- [api-playbook.md](api-playbook.md) - APIs
- [erd-playbook.md](erd-playbook.md) - Data Models

### Agent Profiles (Personas)
- [melissa-ai-spec-whisperer.md](melissa-ai-spec-whisperer.md)
- [hal9000.md](hal9000.md)
- [rosie-x.md](rosie-x.md)
- [bryan-developer-profile.md](bryan-developer-profile.md)

### Templates & Guides (Reference)
- [adr-template.md](adr-template.md)
- [claude-slash-command.md](claude-slash-command.md)
- [dependency-manager-playbook.md](dependency-manager-playbook.md)
- [mermaid-guide.md](mermaid-guide.md)
- [prompt-playbook.md](prompt-playbook.md)
- [rubric-judge.md](rubric-judge.md)

---

## 🚫 What This Repo Is Not

- Not a generic AI prompt library
- Not a wiki
- Not chat memory
- Not speculative documentation
- Not executable code (use for reference only)

If something matters, it becomes an **artifact**.

---

## 📝 Contributing

This is a curated artifact repository. Changes should be:
1. **Tested** - Validated in real usage
2. **Documented** - Clear purpose and usage
3. **Versioned** - Tracked in [CHANGELOG.md](CHANGELOG.md)
4. **Reviewed** - Following the authority model

See [.github/PULL_REQUEST_TEMPLATE.md](.github/PULL_REQUEST_TEMPLATE.md) for contribution guidelines.

---

## 📜 License

This repository is public for reference. See [LICENSE](LICENSE) if present, or assume standard open-source practices.

---

## Final Rule

> **If it isn't written, validated, and versioned — it isn't real.**

This repository exists to make that rule usable at speed.

---

**Repository**: https://github.com/lucebryan3000/melissa.ai_artifacts
**Maintainer**: Luce Bryan
**Version**: See [CHANGELOG.md](CHANGELOG.md)
