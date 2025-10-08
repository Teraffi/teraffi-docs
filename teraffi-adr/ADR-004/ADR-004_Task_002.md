# Task: Publishable Key Validation (HMAC-SHA256 + Rotation) (4-6 hours)
**ADR:** ADR-004  
**Date:** 2025-10-01

## Inputs
- Key Vault client
- Redis cache

## Outputs
- Validator + rotation overlap

## Acceptance Criteria
- [ ] Valid keys accepted
- [ ] Expired rejected
- [ ] Overlap honored

## Implementation Details
- Verify HMAC over timestamp.body; cache secrets; check expiry window.

## Files to Create
- packages/api/src/middleware/publishableKey.ts
- packages/api/test/publishableKey.spec.ts

## Dependencies
Task 004-001
