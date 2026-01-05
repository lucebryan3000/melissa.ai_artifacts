# CI Enforcement Policy Template (Phase-Based Gates)

Each document type must supply a policy file with these sections:

- draft
- p0_problem
- p1_spec
- p2_impl
- p3_staging
- p4_limited
- p5_readout
- p6_ga_or_rollback

## TBD clearing table (keep simple)
Define, by phase, which link fields may remain `TBD` and which must be populated.

Example pattern:
- draft: placeholders OK
- p1_spec: key links exist (may still be TBD if not yet produced)
- p3_staging: api/spec/schema non-TBD
- p6_ga_or_rollback: runbook/threat_model non-TBD
