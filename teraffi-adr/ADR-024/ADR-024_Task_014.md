# Task: System Health Dashboard (4-5 hours)
**ADR:** ADR-024  
**Date:** 2025-10-01

## Inputs
- Infrastructure metrics
- Service health checks

## Outputs
- System overview dashboard
- Resource utilization
- Service status

## Acceptance Criteria
- [ ] CPU utilization per host
- [ ] Memory usage per host
- [ ] Disk space per host
- [ ] Network throughput
- [ ] Service status (up/down)
- [ ] Container health (if using Docker/K8s)
- [ ] Alert status overview
- [ ] System load average

## Implementation Details
Create infrastructure overview dashboard. Show all hosts, resource utilization, service status. Quick view of overall system health.

## Files to Create
- infrastructure/grafana/dashboards/system-health.json
- infrastructure/grafana/dashboards/infrastructure-overview.json

## Dependencies
Task 024-002, Task 024-010