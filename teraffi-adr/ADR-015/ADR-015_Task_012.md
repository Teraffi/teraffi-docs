# Task: Observability & Metrics (3-4 hours)
**ADR:** ADR-015  
**Date:** 2025-10-01

## Inputs
- Metrics from affinity engine
- Component scores
- Processing times

## Outputs
- Metrics collection
- Grafana dashboards
- Prometheus alerts

## Acceptance Criteria
- [ ] Track processing times (first-time vs cached)
- [ ] Track component scores distribution
- [ ] Track cache hit rates
- [ ] Monitor triple extraction success/failure
- [ ] Grafana dashboard for affinity metrics
- [ ] Alerts for slow processing or failures
- [ ] Cost tracking (LLM usage)

## Implementation Details
Emit metrics at each stage of affinity calculation. Create dashboard showing processing performance, score distributions, cache efficiency. Set up alerts for anomalies.

## Files to Create
- packages/affinity-engine/src/metrics.ts
- infrastructure/grafana/dashboards/affinity-engine.json
- infrastructure/prometheus/alerts/affinity-alerts.yml

## Dependencies
Task 015-009, Monitoring infrastructure