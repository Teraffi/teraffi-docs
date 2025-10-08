# Task: SQL Lint Gate for Tenant Predicates (2-3 hours)
**ADR:** ADR-006  
**Date:** 2025-10-01

## Inputs
- Sqlfluff or custom

## Outputs
- CI script

## Acceptance Criteria
- [ ] Fails on missing tenant filters
- [ ] Reports offending lines
- [ ] Docs updated

## Implementation Details
- Add lint rule; integrate into CI; doc exemptions.

## Files to Create
- scripts/ci/sql-tenant-check.ts
- .github/workflows/sql.yml

## Dependencies
Task 006-002
