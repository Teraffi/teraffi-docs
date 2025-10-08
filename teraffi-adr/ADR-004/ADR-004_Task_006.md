# Task: Auth/Keys Integration Test Suite (6-8 hours)
**ADR:** ADR-004  
**Date:** 2025-10-01

## Inputs
- API + Key Vault mock

## Outputs
- E2E tests

## Acceptance Criteria
- [ ] End-to-end passes
- [ ] Replay attack blocked
- [ ] Bad key rejected

## Implementation Details
- Consolidate granular tests into full-path suite.

## Files to Create
- packages/api/test/auth.e2e.spec.ts

## Dependencies
Tasks 004-001..005
