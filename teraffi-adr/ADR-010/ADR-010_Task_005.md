# Task: Retention Policy Config & Docs (2-3 hours)
**ADR:** ADR-010  
**Date:** 2025-10-01

## Inputs
- Policy table

## Outputs
- Docs

## Acceptance Criteria
- [ ] Table mapping published
- [ ] CI enforces policy presence
- [ ] Runbook included

## Implementation Details
- Publish retention matrix and add CI to require entries.

## Files to Create
- docs/data/retention_matrix.md
- scripts/ci/check-retention.ts

## Dependencies
Tasks 010-001..004
