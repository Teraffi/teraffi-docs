# Task: Idempotency Table DDL & Orders Unique Index (2-3 hours)
**ADR:** ADR-007  
**Date:** 2025-10-01

## Inputs
- Migration tool

## Outputs
- SQL migrations

## Acceptance Criteria
- [ ] Table created
- [ ] Unique partial index
- [ ] Rollback script

## Implementation Details
- Add `idempotency_keys` table; unique index on orders.

## Files to Create
- db/migrations/001_idempotency.sql
- db/migrations/004_orders_idx.sql

## Dependencies
None
