# Document Type Creation Standard (Core + Profiles + Registry)

Version: 1.2
Status: active

This standard defines how to create and govern document types so they scale from 4 → 13 without drift.

## Core 10 Rules (Inherited by All Document Types)

1) Canonical format
- Canonical source is Markdown + YAML frontmatter by default.
- Rendered variants (PDF/DOCX) are derived and non-authoritative.

2) Canonical storage and naming
- Each doc type declares a canonical directory and filename pattern.
- Tools/CI discover docs via path + pattern.

3) Required artifact set (minimum viable document type)
- Document Type Spec
- Canonical Template (authoring source)
- Frontmatter JSON Schema (lintable contract)
- CI Enforcement Policy (phase-based gates)

4) Global status taxonomy
- draft, p0_problem, p1_spec, p2_impl, p3_staging, p4_limited, p5_readout, p6_ga_or_rollback

5) Frontmatter (or meta) is the machine contract
- Required keys exist and validate against schema.
- Profiles may add requirements; they do not remove the core base requirements.

6) Body is the execution contract
- Required headings/sections exist in the Markdown body.
- Type profiles define required headings.

7) Fail-closed
- Missing required keys or schema violations = invalid document.
- CI fails when current-phase requirements are not met.

8) CI phase gates exist
- Each doc type provides a CI policy that tightens requirements by phase.

9) Stable IDs
- Once a PR is opened, IDs must not change.
- Only append new IDs; do not rename existing IDs.

10) Versioning / breaking changes
- Breaking changes require:
  - version bump in the Document Type Spec and registry type entry
  - migration note (how to update existing docs)

---

## Core Base Contract (Logical Keys)

For formats that permit a metadata block, the logical core keys are:

- doc_type (e.g., prd, frd, techspec)
- doc_id (e.g., FT-123, FRD-123, TS-123)
- name
- why_now
- status
- decision_freeze_phase (0–6)
- owner
- backup_owner
- links (object; required keys are type-specific but keys must exist)
- gate_summary (object; type-specific contents)
- decisions (array of decision log entries)

Compatibility rule:
- Existing types may use prd_id / frd_id / techspec_id as aliases for doc_id until migrated.

---

## Non-Markdown Types (JSON/JSONC/YAML)

- JSON/JSONC types must include a top-level meta object containing the same logical core keys.
- $schema is recommended in JSON/JSONC/YAML for tooling.
- YAML may be used as an authoring replacement for JSON; emit JSON for strict interchange if needed.

---

## Registry (Canonical Index)

Registry is canonical for:
- discovery (where docs live, how to find them)
- pointers to artifacts (spec/template/schema/policy/profile)
- tool hooks (lint/render/score commands) and CI job name
- pointers to rubrics and health-flag definitions

Registry is minimal: it does not embed enforcement text—only pointers.

---

## Scoring (Non-blocking, 1–10)

- Every doc type has scoring.
- Score is informational (non-blocking).
- Overall score is a weighted average of Core 6 subscores (0–10 each):
  1) Structure (10%)
  2) Contract Completeness (25%)
  3) Testability (20%)
  4) Operational Readiness (15%)
  5) Risk & Safety (20%)
  6) Traceability (10%)

Rubric storage:
- Core: rubrics/core.yaml
- Per-type override: rubrics/<type>.yaml

---

## Health Flags (Non-blocking)

Health flags provide high-signal warnings:
- Core health flags (shared across all types)
- Per-type health flags (additions)

Severity:
- WARN
- CRITICAL

---

## Minimal Governance

To add a new document type:
- PR must include baseline artifacts + registry entry + profile + rubric override + health flag additions
- One reviewer from standards owners approves.
