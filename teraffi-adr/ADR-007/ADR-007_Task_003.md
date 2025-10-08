# Task: Operation-In-Progress 409 Policy + Retry-After (2-3 hours)
**ADR:** ADR-007  
**Date:** 2025-10-01

## Inputs
- Middleware

## Outputs
- Policy code + tests

## Acceptance Criteria
- [ ] 409 emitted
- [ ] Retry-After: 5s header
- [ ] Unit tests pass

## Implementation Details
- Return 409 when response is NULL; document client guidance.

## Files to Create
- packages/api/src/middleware/idempotency.ts
- docs/api/client_retries.md

## Dependencies
Task 007-001
