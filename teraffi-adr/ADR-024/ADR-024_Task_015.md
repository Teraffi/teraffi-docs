# Task: Alert Rules Configuration (6-8 hours)
**ADR:** ADR-024  
**Date:** 2025-10-01

## Inputs
- SLA requirements
- Incident patterns
- Alerting best practices

## Outputs
- Prometheus alert rules
- Alert severity definitions
- Alert documentation

## Acceptance Criteria
- [ ] High error rate alert (>5% for 5 min)
- [ ] High latency alert (p95 >1s for 10 min)
- [ ] Service down alert (2 min)
- [ ] Database connection pool alert (>80%)
- [ ] Disk space low alert (<10%)
- [ ] High database latency alert
- [ ] Business metric anomaly alerts
- [ ] Alert severity levels (critical, warning, info)
- [ ] Runbook links in annotations

## Implementation Details
Define alert rules in Prometheus. Set appropriate thresholds and durations. Include runbook links in annotations. Test alerts fire correctly.

## Files to Create
- infrastructure/prometheus/alerts/api-alerts.yml
- infrastructure/prometheus/alerts/database-alerts.yml
- infrastructure/prometheus/alerts/business-alerts.yml
- infrastructure/prometheus/alerts/infrastructure-alerts.yml
- docs/runbooks/alert_response.md

## Dependencies
Task 024-001