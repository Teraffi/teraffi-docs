# Task: Orders Archival Flag & Snapshot (3-4 hours)
**ADR:** ADR-010  
**Date:** 2025-10-01

## Inputs
- orders table

## Outputs
- Archival script

## Acceptance Criteria
- [ ] archived_at set
- [ ] Snapshot saved
- [ ] No FK breakage

## Implementation Details
- Add archived_at and snapshot export; keep rows in DB.

## Files to Create
- db/migrations/013_orders_archived_at.sql
- packages/ops/src/jobs/orders_snapshot.ts

## Dependencies
ADR-010
