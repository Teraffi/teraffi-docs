# Task: Real-time Materialized Views (4-5 hours)
**ADR:** ADR-018  
**Date:** 2025-10-01

## Inputs
- ClickHouse capabilities
- Real-time requirements
- Query patterns

## Outputs
- Materialized views
- Real-time aggregations
- Refresh strategies

## Acceptance Criteria
- [ ] mv_daily_partnership_metrics
- [ ] mv_monthly_metrics
- [ ] mv_affinity_score_snapshots
- [ ] mv_trend_momentum_realtime
- [ ] Automatic refresh on data insertion
- [ ] Partitioning for performance
- [ ] Query performance <1 second
- [ ] Documentation

## Implementation Details
Create ClickHouse materialized views for common aggregations. Use SummingMergeTree or AggregatingMergeTree engines. Configure automatic refresh. Partition by date for efficient querying.

## Files to Create
- analytics/clickhouse/materialized-views/partnership_metrics.sql
- analytics/clickhouse/materialized-views/monthly_rollups.sql
- docs/analytics/materialized_views.md

## Dependencies
Task 018-002