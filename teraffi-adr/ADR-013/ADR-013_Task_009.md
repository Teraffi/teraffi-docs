# Task: GraphRAG Observability & Cost Tracking (3-4 hours)
**ADR:** ADR-013  
**Date:** 2025-10-01

## Inputs
- Metrics from each pipeline step
- Token usage from LLM
- Query performance data

## Outputs
- Metrics collection
- Cost tracking
- Performance dashboards
- Alerts

## Acceptance Criteria
- [ ] Track latency per pipeline step
- [ ] Track LLM token usage and costs
- [ ] Monitor cache hit rates
- [ ] Track confidence scores
- [ ] Grafana dashboard
- [ ] Alerts for high costs or slow queries
- [ ] Daily cost reports

## Implementation Details
Emit metrics at each pipeline step. Calculate costs from token usage. Create dashboard showing query performance, cache efficiency, daily costs. Set up alerts for anomalies.

## Files to Create
- packages/graphrag-service/src/metrics.ts
- packages/graphrag-service/src/cost-calculator.ts
- infrastructure/grafana/dashboards/graphrag.json
- infrastructure/prometheus/alerts/graphrag-alerts.yml

## Dependencies
Task 013-006, Monitoring infrastructure