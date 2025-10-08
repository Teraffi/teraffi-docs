# Task: Retry/Timeout Policy Module (2-3 hours)
**ADR:** ADR-002  
**Date:** 2025-10-01

## Inputs
- Policy specs

## Outputs
- Shared policy module

## Acceptance Criteria
- [ ] 200ms→30s backoff
- [ ] maxAttempts 7
- [ ] Checkout TTL ~15m

## Implementation Details
- Export typed retry configs; import from workflows.

## Files to Create
- packages/worker/src/config/retryPolicies.ts

## Dependencies
Task 002-001
