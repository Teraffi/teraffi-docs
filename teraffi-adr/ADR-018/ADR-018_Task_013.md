# Task: Observability & Performance Monitoring (3-4 hours)
**ADR:** ADR-018  
**Date:** 2025-10-01

## Inputs
- ClickHouse metrics
- ETL pipeline metrics
- Query performance data

## Outputs
- Metrics collection
- Dashboards for analytics infrastructure
- Alerts

## Acceptance Criteria
- [ ] Track ETL pipeline success/failure rates
- [ ] Track data freshness lag
- [ ] Monitor ClickHouse query performance
- [ ] Track dashboard usage statistics
- [ ] Monitor disk usage growth
- [ ] Grafana dashboard for analytics ops
- [ ] Alerts for pipeline failures
- [ ] Alerts for query performance degradation

## Implementation Details
Instrument ETL pipeline with metrics. Monitor ClickHouse performance metrics. Track data freshness. Create operational dashboard for data team. Alert on failures or performance issues.

## Files to Create
- analytics/monitoring/metrics.py
- infrastructure/grafana/dashboards/analytics-ops.json
- infrastructure/prometheus/alerts/analytics-alerts.yml

## Dependencies
Task 018-006, Monitoring infrastructure