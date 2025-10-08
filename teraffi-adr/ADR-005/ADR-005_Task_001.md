# Task: LaunchDarkly Integration (API + Worker) (3-4 hours)
**ADR:** ADR-005  
**Date:** 2025-10-01

## Inputs
- LD SDK keys

## Outputs
- Init modules

## Acceptance Criteria
- [ ] Evaluates flags
- [ ] Per-request context
- [ ] Shutdown cleanly

## Implementation Details
- Initialize LD in API/worker; context from tenant/user.

## Files to Create
- packages/api/src/flags/index.ts
- packages/worker/src/flags/index.ts

## Dependencies
ADR-001
