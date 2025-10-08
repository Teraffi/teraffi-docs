# Task: Archival Export to Blob (Parquet) + Mark Archived (4-6 hours)
**ADR:** ADR-010  
**Date:** 2025-10-01

## Inputs
- Blob creds
- SQL/COPY

## Outputs
- Export job

## Acceptance Criteria
- [ ] Parquet written
- [ ] archived_at set
- [ ] Idempotent rerun

## Implementation Details
- Implement export job with verification step.

## Files to Create
- packages/ops/src/jobs/archive_export.ts

## Dependencies
Task 010-001
