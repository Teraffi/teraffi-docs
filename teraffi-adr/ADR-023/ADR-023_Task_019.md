# Task: Billing Observability & Monitoring (3-4 hours)
**ADR:** ADR-023  
**Date:** 2025-10-01

## Inputs
- Billing metrics
- Payment data

## Outputs
- Metrics collection
- Grafana dashboards
- Alerts

## Acceptance Criteria
- [ ] Track payment success rate
- [ ] Track failed payment rate
- [ ] Track subscription churn
- [ ] Track MRR growth
- [ ] Monitor webhook processing
- [ ] Track Stripe API latency
- [ ] Grafana dashboard
- [ ] Alerts for failed payments spike
- [ ] Alerts for webhook processing delays

## Implementation Details
Instrument billing system with metrics. Monitor payment success, failures, churn. Track webhook processing. Create dashboard. Alert on anomalies.

## Files to Create
- packages/billing/src/metrics.ts
- infrastructure/grafana/dashboards/billing.json
- infrastructure/prometheus/alerts/billing-alerts.yml

## Dependencies
Task 023-006