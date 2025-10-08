# Task: Partition Drop Job (18m retention) (2-3 hours)
**ADR:** ADR-010  
**Date:** 2025-10-01

## Inputs
- Scheduler

## Outputs
- Drop script

## Acceptance Criteria
- [ ] Drops only after export
- [ ] Logs actions
- [ ] Alerts on failure

## Implementation Details
- Add safe drop with guard file existence.

## Files to Create
- db/cron/drop_old_partitions.sql

## Dependencies
Task 010-002
