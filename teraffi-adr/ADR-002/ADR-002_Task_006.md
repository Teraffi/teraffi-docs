# Task: Workflow Determinism & Retry Test Suite (6-8 hours)
**ADR:** ADR-002  
**Date:** 2025-10-01

## Inputs
- Both workflows
- Policies

## Outputs
- 30+ test scenarios

## Acceptance Criteria
- [ ] Deterministic state
- [ ] Retries per policy
- [ ] Signals exercised

## Implementation Details
- Consolidate prior granular cases into one suite.

## Files to Create
- packages/worker/test/workflows.det.spec.ts

## Dependencies
Tasks 002-002..005
