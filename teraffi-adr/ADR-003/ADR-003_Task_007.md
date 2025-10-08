# Task: End-to-End Outbox E2E Test Suite (6-8 hours)
**ADR:** ADR-003  
**Date:** 2025-10-01

## Inputs
- DB+relay+bus+consumer

## Outputs
- E2E tests

## Acceptance Criteria
- [ ] No loss/duplication
- [ ] Lag within SLO
- [ ] Publisher updates read model

## Implementation Details
- Consolidate prior granular cases into one suite.

## Files to Create
- packages/ops/test/outbox.e2e.spec.ts

## Dependencies
Tasks 003-001..006
