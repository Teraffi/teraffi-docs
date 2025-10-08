# Task: PostgreSQL to Staging Extract (5-6 hours)
**ADR:** ADR-018  
**Date:** 2025-10-01

## Inputs
- PostgreSQL operational database
- Extract requirements
- Scheduling needs

## Outputs
- Extract scripts
- Incremental load logic
- Change data capture (optional)

## Acceptance Criteria
- [ ] Extract partnerships data
- [ ] Extract deal_documents, comments, activities
- [ ] Extract milestones and KPIs
- [ ] Extract member_goals
- [ ] Incremental extraction based on updated_at
- [ ] Store in staging area (S3 or ClickHouse staging tables)
- [ ] Handle deletes/updates appropriately
- [ ] Run time <15 minutes
- [ ] Error handling and logging

## Implementation Details
Build extraction scripts to pull data from PostgreSQL. Use updated_at timestamps for incremental loads. Store in staging area before transformation. Consider Debezium CDC for real-time streaming if needed.

## Files to Create
- analytics/etl/extract/postgres_extract.py
- analytics/etl/extract/incremental_strategy.py
- analytics/etl/test/test_extract.py

## Dependencies
Task 018-002, ADR-007 (PostgreSQL)