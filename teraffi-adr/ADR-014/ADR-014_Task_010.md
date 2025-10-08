# Task: Observability & Monitoring (3-4 hours)
**ADR:** ADR-014  
**Date:** 2025-10-01

## Inputs
- Metrics from gateway
- Cost data
- Performance data

## Outputs
- Metrics collection
- Grafana dashboards
- Prometheus alerts

## Acceptance Criteria
- [ ] Emit metrics (latency, tokens, cost, cache hit rate)
- [ ] Grafana dashboard for LLM usage
- [ ] Alerts for high cost, high latency, provider down
- [ ] Track by provider, model, use_case
- [ ] Cost projection metrics
- [ ] Error rate monitoring

## Implementation Details
Instrument gateway with OpenTelemetry metrics. Create dashboards showing usage, costs, performance. Define alert rules for anomalies.

## Files to Create
- packages/llm-gateway/src/metrics.ts
- infrastructure/grafana/dashboards/llm-gateway.json
- infrastructure/prometheus/alerts/llm-gateway-alerts.yml
- docs/observability/llm_gateway_metrics.md

## Dependencies
Task 014-008, Monitoring infrastructure