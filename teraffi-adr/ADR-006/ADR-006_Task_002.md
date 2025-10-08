# Task: Guard Integration into Repos & Query Helpers (3-4 hours)
**ADR:** ADR-006  
**Date:** 2025-10-01

## Inputs
- Kysely repos

## Outputs
- Repository wrappers

## Acceptance Criteria
- [ ] All repos import guard
- [ ] Helper used in queries
- [ ] CI scan clean

## Implementation Details
- Wrap repo factories to inject tenant predicate helpers.

## Files to Create
- packages/api/src/repos/*
- scripts/ci/scan-tenant-predicate.ts

## Dependencies
Task 006-001
