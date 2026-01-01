# Melissa.ai Artifacts

> Supporting documentation for **Melissa.ai Spec Whisperer** — the orchestrator that coaxes clarity out of chaos.

---

## What Is This?

This repo contains playbooks, question banks, and templates that extend Melissa's capabilities. Melissa is a spec discovery and refinement protocol for AI-assisted peer coding. She asks targeted questions, locks in decisions, and helps produce high-quality artifacts.

**Core prompt:** [`melissa-ai-spec-whisperer.md`](melissa-ai-spec-whisperer.md)

---

## Quick Start

### Option 1: Use Melissa Standalone
Copy the contents of `melissa-ai-spec-whisperer.md` into your AI assistant (Claude, ChatGPT, Claude Code, Codex CLI). Paste your idea, draft, or spec. Melissa starts asking questions immediately.

### Option 2: Use Melissa with Supplementary Docs
Melissa can fetch playbooks from this repo on-demand for deeper guidance:

```
https://raw.githubusercontent.com/lucebryan3000/melissa.ai_artifacts/main/
```

When you go deep on a specific artifact type, Melissa will pull the relevant playbook to sharpen her questions.

---

## Artifact Index

### Playbooks
Detailed guidance for each artifact type. Includes mental models, evaluation dimensions, question banks, and invariants.

| Playbook | Specialist Role | Status |
|----------|-----------------|--------|
| [`prd-playbook.md`](prd-playbook.md) | Product Thinker | ✅ Ready |
| [`frd-playbook.md`](frd-playbook.md) | Requirements Analyst | ✅ Ready |
| [`architecture-playbook.md`](architecture-playbook.md) | Systems Architect | ✅ Ready |
| [`techspec-playbook.md`](techspec-playbook.md) | Solutions Architect | ✅ Ready |
| [`migration-playbook.md`](migration-playbook.md) | Risk Analyst | ✅ Ready |
| [`mermaid-guide.md`](mermaid-guide.md) | Visual Clarity Reviewer | ✅ Ready |
| [`erd-playbook.md`](erd-playbook.md) | Data Modeler | ✅ Ready |
| [`prompt-playbook.md`](prompt-playbook.md) | Prompt Engineer | ✅ Ready |
| [`api-playbook.md`](api-playbook.md) | Integration Specialist | ✅ Ready |
| [`adr-template.md`](adr-template.md) | Decision Analyst | ✅ Ready |
| [`implementation-playbook.md`](implementation-playbook.md) | Delivery Planner | ✅ Ready |

### Templates
Structural skeletons for artifact production. Coming soon.

### Question Banks
Extended question sets for deep-dives. Coming soon.

---

## How Melissa Uses This Repo

1. **Standalone mode:** Melissa works without fetching anything. The core prompt has embedded evaluation dimensions for all artifact types.

2. **Deep-dive mode:** When going deep on a specialist domain, Melissa fetches the relevant playbook:
   ```
   https://raw.githubusercontent.com/lucebryan3000/melissa.ai_artifacts/main/prompt-playbook.md
   ```

3. **Artifact production:** When you request an artifact, Melissa can fetch templates to ensure structural consistency.

---

## Playbook Structure

All playbooks follow a consistent structure (established by [Dependency Manager Playbook](Dependency%20Manager%20Playbook.md)):

```
# [Artifact] Playbook

> **Purpose**: [One-line description]
> **Version**: X.X
> **Last Updated**: YYYY-MM-DD

---

## Mental Model
[Visual/conceptual overview of the artifact]

## Inputs / Outputs
[What goes in, what comes out]

## Evaluation Dimensions
[Quality criteria with probing questions]

## Question Bank
[Extended questions organized by dimension]

## Common Pitfalls
[Anti-patterns and mistakes to catch]

## Template
[Structural skeleton]

## Invariants
[Hard rules that must always hold]

## Examples
[Good/bad examples with annotations]
```

---

## Contributing

This is primarily Bryan's workspace. Artifacts evolve through real usage with Melissa. If you're Bryan:

1. Use Melissa to refine ideas
2. Lock decisions through Q&A rounds
3. Produce artifacts
4. Add them here
5. Iterate

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2024-12-31 | Complete repo: README, core prompt, all 11 specialist playbooks |
| 0.1 | 2024-12-31 | Initial repo: README, core prompt, first 3 playbooks |

---

*Melissa.ai Spec Whisperer — Coaxing clarity out of chaos.*
