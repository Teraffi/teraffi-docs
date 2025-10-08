# Task: Consolidated Archival/Retention Test Suite (4-6 hours)
**ADR:** ADR-010  
**Date:** 2025-10-01

## Inputs
- Exporters + drops

## Outputs
- E2E tests

## Acceptance Criteria
- [ ] Idempotent exports
- [ ] No data loss
- [ ] Restorable snapshots

## Implementation Details
- End-to-end tests for archival lifecycle.

## Files to Create
- packages/ops/test/archival.e2e.spec.ts

## Dependencies
Tasks 010-001..005
