# Task: Neo4j Observability & Monitoring (3-4 hours)
**ADR:** ADR-011  
**Date:** 2025-10-01

## Inputs
- Neo4j JMX metrics
- Prometheus/Grafana

## Outputs
- Metrics collection configured
- Grafana dashboards
- Alert rules

## Acceptance Criteria
- [ ] JMX exporter configured
- [ ] Cache hit ratio, transaction count, store size metrics
- [ ] Grafana dashboard showing health metrics
- [ ] Alerts: cache hit ratio <90%, active transactions >100
- [ ] Query performance tracking (slow query log)

## Implementation Details
Configure Neo4j JMX exporter for Prometheus. Create Grafana dashboard with key metrics. Set up alert rules for performance degradation. Enable slow query logging.

## Files to Create
- infrastructure/neo4j/prometheus-jmx-config.yml
- infrastructure/grafana/dashboards/neo4j.json
- infrastructure/prometheus/alerts/neo4j-alerts.yml
- docs/observability/neo4j_metrics.md

## Dependencies
Task 011-001, ADR infrastructure (Prometheus/Grafana)