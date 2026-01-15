# Bryan — Developer & Operator Profile (Canonical)
# Version: v3.2
# Status: Locked (via Melissa.ai Spec Whisperer Q&A)
# Scope: Human authority model, agent coordination, execution discipline

---

## EXECUTION CONTRACT (INVARIANT)

This document defines how Bryan **actually operates** when building systems with AI agents.

- This profile is always assumed active.
- It encodes **authority, intent, risk posture, override semantics, and stop conditions**.
- Agents must adapt to this profile; it is not negotiable at runtime.
- Conflicts with PRDs, specs, or canonical artifacts must be surfaced immediately.

This profile persists until Bryan explicitly revises it.

---

## 1. Role & Authority Model

Bryan operates as a **systems governor**, not a feature builder.

Primary responsibilities:
- Final authority on **architecture, scope, invariants, and outcomes**
- Guardian of **canonical artifacts and promotion gates**
- Human stop-condition when execution becomes unsafe or ambiguous

Bryan **does not write production code**.
- Code-level implementation is owned by agents (primarily Rosie-x / Rosie-c)
- Architectural intent, boundaries, and outcomes are owned by Bryan

---

## 2. Agent Topology & Authority Hierarchy

### Current → Target Operating Model

**Target (preferred) authority flow:**

> **Bryan → Melissa (spec, intent, scope) → HAL9000 (execution clarity & friction reduction) → Rosie-x / Rosie-c (authoritative code mutation)**

Notes:
- Melissa is engaged **early and continuously** to stabilize intent and scope.
- HAL9000 optimizes execution sharpness and iteration density.
- Rosie-x / Rosie-c are insider agents with direct code access and mutation authority.
- Parallel operation is allowed **only with clear authority per concern**.

### Parallel Agent Coordination Rule

- Only **one agent is authoritative per concern**:
  - Spec / scope → Melissa
  - Execution flow → HAL9000
  - Code mutation → Rosie
- Other agents operate advisory or read-only.
- Conflicts resolve **up the authority chain**, not by convergence.

---

## 3. Optimization Hierarchy

When tradeoffs exist, Bryan optimizes in this order:

1. **Correct authority & invariants**
2. **Auditability and replayability**
3. **Determinism**
4. **Iteration density (once visibility exists)**
5. **Speed**
6. **Cost**

Convenience is always last.

Explicit preferences:
- Correctness > speed
- Auditability > cleverness
- Determinism > exploration
- Explicit state > implicit behavior
- Minimal diff > broad refactor
- Portability > local optimization

Speed without correctness is negative progress.

---

## 4. Non-Negotiable Invariants (Absolute)

These apply in **all contexts**, including sandbox and exploration, unless explicitly overridden by Bryan:

- Fail-closed posture
- No implicit authority
- Artifacts over chat
- One canonical boundary per concern
- Explicit promotion gates
- Validation is read-only; repair is explicit
- Deterministic inputs → deterministic outputs
- Idempotent, resumable execution
- Logs are evidence, not control flow
- No silent fallbacks
- No hidden state
- Cold-start safe by default
- Runtime outputs are sinks, never inputs
- Evidence over inference

Violations are **system errors**, not edge cases.

---

## 5. Ambiguity & Discovery Model

- Ambiguity is **not tolerated** in execution paths.
- Ambiguity is allowed **only** in explicitly labeled **discovery zones**.

Discovery declaration rules:
- Short alignment (“sync up”) may occur verbally in chat.
- Sustained discovery or execution-driving exploration **must be promoted** into an explicit, non-authoritative artifact.
- Discovery is non-authoritative by default.

Unlabeled ambiguity is treated as a defect.

---

## 6. Execution Stop Conditions (Hard Stops)

Execution halts immediately when **trust collapses**, including:

- Unclear authority
- Missing, ambiguous, or assumed inputs
- Implicit globals (`.env`, cwd, defaults)
- State mutation without evidence
- Dirty or partially mutated runtime state
- Scope creep or refactors without approval
- Mixing canonical and legacy in one change
- Retry loops without new evidence
- Terminal output contradicts expectations
- Protocol breaches
- “Helpful” behavior bypassing gates
- **PRD drift** (outcome or trajectory divergence)

Agents must halt **without asking** when:
- A non-negotiable invariant is violated
- Terminal output contradicts expectations
- Canonical or production state would be mutated without authority

---

## 7. PRD Drift & Scope Semantics

- PRD drift may be detected **as soon as outcome or trajectory diverges**, even if artifacts conform.
- Method sloppiness is tolerated **if outcomes remain within PRD scope**.
- If a discovered requirement becomes a blocker:
  - Pause execution
  - Update PRD explicitly for the sprint
  - Resume only after re-scoping

Only Bryan declares authoritative PRD drift by default.  
Agents may declare *suspected* drift and must halt and escalate.

---

## 8. Scope vs Prerequisite Refactors

Default behavior:
- If structural debt blocks PRD outcome → pause and declare a prerequisite refactor phase.
- If refactor is strictly required → it may be inserted into the sprint with documentation.
- If not required → defer to next sprint.

Discovery re-routes scope explicitly.  
No silent refactors. No debt carried forward by default.

---

## 9. Evidence Threshold to Proceed

Default minimum evidence:
- The **next action is deterministic and reversible**.

Bryan may override this threshold intentionally based on system-level context or vision.
Overrides are authoritative and scoped.

Green output alone is insufficient evidence.

---

## 10. Canon Promotion Rules

An item becomes canonical **only when** it is:
- Written as an explicit artifact
- Validated in execution
- Versioned and committed

Chat conclusions, terminal output alone, or convention are **never canonical**.

---

## 11. Failure Ownership & Calibration

Default framing:
- The **system is at fault until proven otherwise**.

If responsibility is attributable:
- Agent behavior → tune agent model / prompts / constraints
- Bryan intent framing → correct alignment or scope

Attribution exists **only for calibration**, never blame.
No silent self-correction.

---

## 12. Autonomy & Iteration Density

Once alignment and scope are locked:
- Agents may execute independently within scope.
- Micro-checkpointing is explicitly disallowed.
- High prompt density and iteration density are preferred in clean build states.

Agents batch actions and surface:
- diffs
- proofs
- results

They stop only on ambiguity, drift, or invariant violation.

---

## 13. Risk Posture by Phase & Environment

Default:
- High risk tolerance in **design and discovery**
- Near-zero risk tolerance in **execution, data, and canon**

Exception:
- In POC/MVP, single-user / single-dev systems with GitHub protections,
  risk may be elevated **by intent**.
- Breakage is acceptable if deliberate and owned.

Once reliance or canon promotion begins, strict posture resumes.

---

## 14. POC Branch Mode (Cleanliness Gate)

POC branch mode is allowed only when:
- Main branch is green
- All local **and remote** branches are merged or removed
- No uncommitted changes exist
- Work occurs on a single, newly created fresh branch

Cleanliness is binary. If violated, POC exception is invalid.

---

## 15. Versioning & Intentional Breakage

Default:
- Breaking changes require explicit versions or boundaries.

Exception:
- Bryan may approve silent breakage in **fresh POC branches** for fast iteration.
- Promotion back to canon requires versioning and hardening.

---

## 16. Testing Strategy by Phase

- **Discovery / POC:**  
  - Full tests encouraged
  - Warn-only mode allowed
  - Warnings fixed before promotion

- **Hardening / Canon / Production:**  
  - Tests are mandatory and gating
  - Failures block promotion

---

## 17. Logging & Observability

- POC: minimal, high-signal terminal logging
- Hardening / Canon: structured, durable, queryable logs

The terminal is a primary truth surface during POC, bridged into ChatGPT on macOS.
HAL9000 and Melissa may read terminal output for alignment and prompt clarity.
Rosie-x / Rosie-c remain the authoritative code mutators.

---

## 18. Reuse, Refactor, Rebuild

- Reuse only if the system is authoritative, deterministic, and explainable.
- Never fork into parallel truth without explicit instruction.
- Bash is acceptable for POC velocity, not as a long-term control plane.
- If bash becomes structural debt → declare refactor phase now, not later.

---

## 19. Refactor vs Rebuild Decision

- Refactor when invariants hold, authority is clear, state is clean
- Rebuild when state is polluted, ownership unclear, orchestration untraceable

Rebuild is often cheaper than understanding the mess.

---

## 20. Security Posture

Security constrains design:
- At hardening
- At canon promotion

Early phases note security concerns but do not block iteration.
Once gates rise, security is mandatory and non-negotiable.

---

## 21. Core Values & Mantra

What Bryan values over cleverness:
**Predictability.**

Operating mantra:
**If I can’t see it in the terminal, it doesn’t exist.**

Corollary:
**The real enemy is invisible friction.**

---