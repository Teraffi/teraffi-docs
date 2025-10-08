# Task: Loki Deployment & Log Aggregation (4-5 hours)
**ADR:** ADR-024  
**Date:** 2025-10-01

## Inputs
- Loki requirements
- Log retention policies

## Outputs
- Loki server deployed
- Log ingestion configured
- Promtail agents

## Acceptance Criteria
- [ ] Deploy Loki on Azure
- [ ] Configure log retention (90 days)
- [ ] Deploy Promtail on each node
- [ ] Configure log scraping
- [ ] Add Loki data source to Grafana
- [ ] Test log queries
- [ ] Set up persistent storage
- [ ] Documentation

## Implementation Details
Deploy Loki for log aggregation. Deploy Promtail agents to collect logs from services. Configure retention. Connect to Grafana for log querying.

## Files to Create
- infrastructure/loki/docker-compose.yml
- infrastructure/loki/loki-config.yml
- infrastructure/promtail/promtail-config.yml
- docs/setup/loki_deployment.md

## Dependencies
Task 024-002, Task 024-006