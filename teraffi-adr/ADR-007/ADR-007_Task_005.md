# Task: Reconciliation Job for Stale Idempotency Rows (2-3 hours)
**ADR:** ADR-007  
**Date:** 2025-10-01

## Inputs
- DB repo

## Outputs
- Job script

## Acceptance Criteria
- [ ] Find >90d rows
- [ ] Archive or purge
- [ ] Report generated

## Implementation Details
- Implement cleanup job with safety window.

## Files to Create
- packages/ops/src/jobs/idem_cleanup.ts

## Dependencies
Task 007-002
