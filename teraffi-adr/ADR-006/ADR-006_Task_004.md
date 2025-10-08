# Task: Index Verification Script (tenant-first) (2-3 hours)
**ADR:** ADR-006  
**Date:** 2025-10-01

## Inputs
- DB introspection

## Outputs
- Script + report

## Acceptance Criteria
- [ ] Ensures indexes on tenant columns
- [ ] CI gate passes
- [ ] Report artifact stored

## Implementation Details
- Query pg_catalog to verify composite indexes; fail on missing.

## Files to Create
- scripts/ci/index-verify.ts

## Dependencies
Task 006-003
