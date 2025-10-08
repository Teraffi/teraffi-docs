# Task: Reconciliation Job for Stuck Orders (3-5 hours)
**ADR:** ADR-002  
**Date:** 2025-10-01

## Inputs
- DB repo
- Temporal client

## Outputs
- Recon script + schedule

## Acceptance Criteria
- [ ] Find stuck states
- [ ] Resumes or alerts
- [ ] E2E test

## Implementation Details
- Query DB for partial states; resume or page on-call.

## Files to Create
- packages/worker/src/jobs/reconcile.ts

## Dependencies
ADR-007, ADR-008
