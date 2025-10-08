# Task: Dimensional Model Design & Implementation (6-8 hours)
**ADR:** ADR-018  
**Date:** 2025-10-01

## Inputs
- Business requirements
- Source data schemas
- Kimball methodology

## Outputs
- Dimension tables (date, entity, trend, stage)
- Fact tables (partnerships, affinity, events, metrics)
- DDL scripts

## Acceptance Criteria
- [ ] dim_date table with full calendar
- [ ] dim_entity table with SCD Type 2
- [ ] dim_trend table
- [ ] dim_deal_stage table
- [ ] fact_partnerships table
- [ ] fact_affinity_scores table
- [ ] fact_deal_events table
- [ ] fact_trend_metrics table
- [ ] fact_user_activity table
- [ ] All foreign keys and indexes
- [ ] Documentation of grain and measures

## Implementation Details
Design star schema following Kimball dimensional modeling. Implement slowly changing dimensions (Type 2) for entity history. Define fact table grains. Create appropriate indexes for query performance.

## Files to Create
- analytics/clickhouse/schemas/dimensions.sql
- analytics/clickhouse/schemas/facts.sql
- analytics/clickhouse/schemas/indexes.sql
- docs/analytics/dimensional_model.md

## Dependencies
Task 018-001