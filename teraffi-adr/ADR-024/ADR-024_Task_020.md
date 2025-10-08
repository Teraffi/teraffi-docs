# Task: Query Performance Profiling (4-5 hours)
**ADR:** ADR-024  
**Date:** 2025-10-01

## Inputs
- Database query traces
- Performance baselines

## Outputs
- Query profiling setup
- Slow query detection
- Performance reports

## Acceptance Criteria
- [ ] Enable PostgreSQL query logging
- [ ] Configure pg_stat_statements
- [ ] Track slow queries (>1s)
- [ ] Create slow query dashboard
- [ ] Alert on query performance degradation
- [ ] Provide query optimization recommendations
- [ ] Integration with tracing

## Implementation Details
Enable query performance tracking in PostgreSQL. Log slow queries. Create dashboard showing slowest queries. Alert when queries degrade.

## Files to Create
- infrastructure/prometheus/exporters/pg-custom-queries.yml
- infrastructure/grafana/dashboards/query-performance.json

## Dependencies
Task 024-009