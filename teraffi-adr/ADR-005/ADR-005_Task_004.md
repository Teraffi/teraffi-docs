# Task: Feature Guard Wiring (Sample Endpoint) (2-3 hours)
**ADR:** ADR-005  
**Date:** 2025-10-01

## Inputs
- API route
- LD eval

## Outputs
- Guarded route

## Acceptance Criteria
- [ ] Disabled → 404
- [ ] Enabled → 200
- [ ] E2E tested

## Implementation Details
- Add feature-guarded route and tests.

## Files to Create
- packages/api/src/routes/feature.example.ts
- packages/api/test/feature.guard.spec.ts

## Dependencies
Tasks 005-001..003
