# Task: Time-Series Data Storage (3-4 hours)
**ADR:** ADR-016  
**Date:** 2025-10-01

## Inputs
- Processed signals
- PostgreSQL partitioning strategy

## Outputs
- Time-series schema
- Partitioned tables
- Aggregation queries

## Acceptance Criteria
- [ ] Create trend_signals table with partitioning
- [ ] Monthly partitions for efficient queries
- [ ] Store signal count, engagement, sentiment by timestamp
- [ ] Indexes for trend_name and timestamp
- [ ] Aggregation queries for daily/weekly rollups
- [ ] Retention policy (archive >90 days)
- [ ] Integration tests

## Implementation Details
Create partitioned table by timestamp (monthly). Store aggregated signal data. Build efficient queries for momentum calculation. Implement retention policy for old data.

## Files to Create
- db/migrations/XXX_create_trend_signals.sql
- packages/social-listening/src/storage/time-series.ts
- packages/social-listening/test/time-series.spec.ts

## Dependencies
Task 016-004, ADR-007 (PostgreSQL)