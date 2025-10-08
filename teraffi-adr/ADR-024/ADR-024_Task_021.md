# Task: SLO/SLI Tracking (5-6 hours)
**ADR:** ADR-024  
**Date:** 2025-10-01

## Inputs
- SLA commitments
- Service objectives

## Outputs
- SLI definitions
- SLO targets
- Error budget tracking

## Acceptance Criteria
- [ ] Define SLIs (latency, availability, error rate)
- [ ] Set SLO targets (e.g., 99.9% uptime)
- [ ] Calculate error budgets
- [ ] Track burn rate
- [ ] Create SLO dashboard
- [ ] Alert on error budget exhaustion
- [ ] Monthly SLO reports

## Implementation Details
Define Service Level Indicators and Objectives. Track availability, latency, error rates against SLO targets. Calculate error budgets and burn rates.

## Files to Create
- infrastructure/prometheus/recording-rules/slo.yml
- infrastructure/grafana/dashboards/slo-tracking.json
- docs/sre/slo_definitions.md

## Dependencies
Task 024-005, Task 024-015