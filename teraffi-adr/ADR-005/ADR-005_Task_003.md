# Task: Flag Registry + CI Drift Check (3-4 hours)
**ADR:** ADR-005  
**Date:** 2025-10-01

## Inputs
- Code search
- Registry file

## Outputs
- CI check

## Acceptance Criteria
- [ ] Unknown flag usage blocked
- [ ] Registry synced
- [ ] Docs generated

## Implementation Details
- Parse code for flag names; compare with registry; fail on mismatch.

## Files to Create
- docs/flags/registry.json
- scripts/ci/check-flags.ts
- .github/workflows/flags.yml

## Dependencies
Tasks 005-001..002
