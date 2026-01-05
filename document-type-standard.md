# Document Type Creation Standard (Core + Profiles + Registry)

Version: 1.1  
Status: active  
Date: 2026-01-05  

This standard defines how to create and govern **document types** in Atlas/Bloom so they scale from 4 → 13 without drift.

---

## Locked Decisions

1) **Canonical format (default):** Markdown + YAML frontmatter  
2) **Canonical locations:** per-type, with deterministic directory + filename pattern  
3) **Minimum artifact set per document type (baseline):**  
   - Document Type Spec  
   - Canonical Template (authoring source)  
   - Frontmatter JSON Schema (lintable contract)  
   - CI Enforcement Policy (phase-based gates)  
4) **Governance structure:** Core Standard + Type Profiles (overrides)  
5) **Profiles representation:** Markdown profiles **and** a registry YAML index, created together  
6) **Status taxonomy:** Global, phase-aligned across all document types  
   - `draft`, `p0_problem`, `p1_spec`, `p2_impl`, `p3_staging`, `p4_limited`, `p5_readout`, `p6_ga_or_rollback`  
7) **Frontmatter contract:** Core base contract required wherever the format permits a metadata block  
   - For non-frontmatter formats (e.g., JSON), the equivalent is a top-level `meta` object  
8) **Registry:** Canonical source of truth (discovery + pointers)  
9) **Registry detail level:** Minimal + pointers + tooling hooks (no enforcement content embedded)  
10) **Tool hooks in registry:** `lint_command`, `render_command`, `score_command`, `ci_job_name`  
11) **Scoring:** Exists for **all document types**, **non-blocking**, **1–10 scale**, plus **health flags**  
12) **Scoring architecture:** Core Scoring Base + per-type rubric overrides  
13) **Core scoring dimensions (6):** Structure, Contract Completeness, Testability, Operational Readiness, Risk & Safety, Traceability  
14) **Overall score:** Weighted average of Core 6 subscores (weights can be overridden per type)  
15) **Default Core 6 weights (baseline):**  
   - Contract Completeness: 25%  
   - Testability: 20%  
   - Risk & Safety: 20%  
   - Operational Readiness: 15%  
   - Traceability: 10%  
   - Structure: 10%  
16) **Rubric storage:** Core YAML + per-type YAML overrides, referenced from registry  
17) **Health flags:** Core Health Flags + per-type additions (non-blocking)

Open (not yet locked): Core Health Flags list selection.

---

## Core 10 Rules (Inherited by All Document Types)

1) **Canonical format**
   - Canonical source is **Markdown + YAML frontmatter** by default.
   - Rendered variants (PDF/DOCX) are derived and non-authoritative.

2) **Canonical storage and naming**
   - Each type declares:
     - canonical directory (e.g., `docs/prd/`)
     - filename pattern (e.g., `FT-####-<slug>.md`)
   - Tools/CI must discover docs via path + pattern.

3) **Required artifact set**
   - Document Type Spec
   - Canonical Template
   - Frontmatter JSON Schema
   - CI Enforcement Policy

4) **Global status taxonomy**
   - Use the shared phase-aligned statuses:
     - `draft`, `p0_problem`, `p1_spec`, `p2_impl`, `p3_staging`, `p4_limited`, `p5_readout`, `p6_ga_or_rollback`

5) **Frontmatter (or meta) is the machine contract**
   - Required keys exist and validate against schema.
   - Profiles may add requirements; they do not remove core base requirements.

6) **Body is the execution contract**
   - Required headings/sections exist in Markdown body.
   - Type profiles define required headings.

7) **Fail-closed**
   - Missing required keys or schema violations = invalid document.
   - CI must fail when current phase requirements are not met.

8) **CI phase gates exist**
   - Each type provides a CI policy that tightens requirements by phase.

9) **Stable IDs**
   - Once a PR is opened, IDs must not change.
   - Only append new IDs; do not rename existing IDs.

10) **Versioning / breaking changes**
   - Breaking changes to a type require:
     - version bump in the Document Type Spec
     - migration note (how to update existing docs)

---

## Core Base Contract Across Formats

### Markdown types
- Use YAML frontmatter as the authoritative machine contract.

### Non-Markdown types (e.g., JSON)
- Use the same logical keys in a top-level `meta` object.
- JSONC may include comments, but `meta` is authoritative.

### YAML as JSON replacement (authoring)
- YAML is allowed as a replacement for JSON where appropriate (authoring/maintenance).
- If you need strict interchange, emit JSON from YAML, but keep the logical contract consistent.

---

## Required Artifacts and Where They Live

For each document type `<type>`:

- Document Type Spec  
  - `docs/standards/document-types/<type>.spec.md`
- Canonical Template  
  - `docs/standards/document-types/templates/<type>.template.md`
- Frontmatter JSON Schema  
  - `docs/standards/document-types/schemas/<type>.frontmatter.schema.json`
- CI Enforcement Policy  
  - `docs/standards/document-types/policies/<type>.ci-policy.md`
- Profile Markdown  
  - `docs/standards/document-types/profiles/<type>.md`
- Registry entry  
  - `docs/standards/document-types/document-types-registry.yaml`

(Exact paths can vary, but must be consistent and registered.)

---

## Type Profiles (Overrides Only)

Profiles inherit Core 10 rules and declare deltas:

- Canonical storage + naming (dir/pattern)
- Type-specific required frontmatter keys (additions)
- Required body headings
- Phase gate deltas
- Type-specific scoring rubric pointer
- Type-specific health flags (additions)

Canonical profile representation:
- `docs/standards/document-types/profiles/<type>.md`

---

## Registry (Canonical Source of Truth)

Registry is canonical and minimal: discovery + pointers + tooling hooks.

Each `types[]` entry MUST include:
- `key`
- `prefix`
- `canonical_dir`
- `filename_pattern`
- `profile` (path)
- `artifacts.document_type_spec`
- `artifacts.canonical_template`
- `artifacts.frontmatter_json_schema`
- `artifacts.ci_enforcement_policy`
- `lint_command`
- `render_command`
- `score_command`
- `ci_job_name`
- `rubric` (path to per-type rubric override YAML)

Registry does NOT embed enforcement rules; it points to the profile + CI policy.

---

## Scoring Standard (Non-Blocking, 1–10)

### Core scoring model
- Each doc gets a **0–10 subscore** per Core 6 dimension:
  1) Structure
  2) Contract Completeness
  3) Testability
  4) Operational Readiness
  5) Risk & Safety
  6) Traceability

### Roll-up score
- Overall score (1–10) = **weighted average** of the six subscores
- Default weights (baseline):
  - Contract Completeness 25%
  - Testability 20%
  - Risk & Safety 20%
  - Operational Readiness 15%
  - Traceability 10%
  - Structure 10%
- Types may override weights in their rubric override YAML.

### Rubric storage
- Core rubric base:
  - `docs/standards/document-types/rubrics/core.yaml`
- Per-type rubric override:
  - `docs/standards/document-types/rubrics/<type>.yaml`
- Registry references the per-type rubric path; scorer applies core + override.

---

## Health Flags (Non-Blocking)

- Health flags are **high-signal warnings** for critical gaps.
- Standard model:
  - **Core health flags** (shared across all types)
  - **Per-type health flags** (additions only)

Open item:
- Core health flags list not yet locked.

---

## Change Protocol

- Any change to a document type contract that breaks existing documents requires:
  - bump version in the Document Type Spec
  - update schema/template/profile/policy as needed
  - write a migration note (how to update existing docs)
  - update registry pointers if filenames/paths changed
