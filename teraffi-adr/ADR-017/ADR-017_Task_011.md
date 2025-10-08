# Task: Observability & Metrics (3-4 hours)
**ADR:** ADR-017  
**Date:** 2025-10-01

## Inputs
- Deal flow metrics
- State transitions
- Processing times

## Outputs
- Metrics collection
- Grafana dashboards
- Performance alerts

## Acceptance Criteria
- [ ] Track deals by stage (funnel metrics)
- [ ] Track transition times (stage duration)
- [ ] Track proposal generation success/time
- [ ] Track contract signing time
- [ ] Monitor milestone completion rates
- [ ] Grafana dashboard
- [ ] Alerts for stalled deals
- [ ] Conversion rate tracking

## Implementation Details
Emit metrics at each stage transition. Track time spent in each stage. Monitor success rates. Create dashboard showing deal funnel and conversion metrics. Alert on deals stalled >30 days.

## Files to Create
- packages/deal-flow/src/metrics.ts
- infrastructure/grafana/dashboards/deal-flow.json
- infrastructure/prometheus/alerts/deal-flow-alerts.yml

## Dependencies
Task 017-008, Monitoring infrastructure