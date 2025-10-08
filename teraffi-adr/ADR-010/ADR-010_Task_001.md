# Task: Monthly Partitioning Migrations (events, webhooks, outbox) (4-6 hours)
**ADR:** ADR-010  
**Date:** 2025-10-01

## Inputs
- DDL

## Outputs
- Migrations

## Acceptance Criteria
- [ ] Partitions created
- [ ] Insert routing works
- [ ] Pruning verified

## Implementation Details
- Create partitioned tables and helpers.

## Files to Create
- db/migrations/010_events_partitioning.sql
- db/migrations/011_webhooks_partitioning.sql
- db/migrations/012_outbox_partitioning.sql

## Dependencies
DB ready
