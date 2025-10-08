# Task: API Performance Dashboard (5-6 hours)
**ADR:** ADR-024  
**Date:** 2025-10-01

## Inputs
- API metrics
- Performance requirements

## Outputs
- API overview dashboard
- Request/error visualizations
- Latency charts

## Acceptance Criteria
- [ ] Request rate panel (by method, route)
- [ ] Error rate panel (by status code)
- [ ] Latency panel (p50, p95, p99)
- [ ] Active connections gauge
- [ ] Request distribution heatmap
- [ ] Top slowest endpoints
- [ ] Error breakdown by endpoint
- [ ] Template variables (environment, service)
- [ ] Alerts configured

## Implementation Details
Create Grafana dashboard visualizing API performance. Show request rates, errors, latency percentiles. Add alerts for anomalies.

## Files to Create
- infrastructure/grafana/dashboards/api-overview.json
- infrastructure/grafana/dashboards/api-performance.json

## Dependencies
Task 024-002, Task 024-005