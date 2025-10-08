# Task: Flagged Rollout E2E Suite (4-6 hours)
**ADR:** ADR-005  
**Date:** 2025-10-01

## Inputs
- API + worker

## Outputs
- E2E tests

## Acceptance Criteria
- [ ] Percent & tenant overrides pass
- [ ] Fallback path covered
- [ ] CI green

## Implementation Details
- Consolidate granular flag tests into one suite.

## Files to Create
- packages/api/test/flags.e2e.spec.ts

## Dependencies
Tasks 005-001..004
