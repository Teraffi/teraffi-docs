# Task: Node Exporter & System Metrics (3-4 hours)
**ADR:** ADR-024  
**Date:** 2025-10-01

## Inputs
- Server inventory
- System metric requirements

## Outputs
- Node exporter deployed
- System metrics collection
- Host-level monitoring

## Acceptance Criteria
- [ ] Deploy Node exporter on all hosts
- [ ] Collect CPU, memory, disk, network metrics
- [ ] Configure filesystem monitoring
- [ ] Add to Prometheus scrape config
- [ ] Test metric collection
- [ ] Documentation

## Implementation Details
Deploy Prometheus Node Exporter on each server. Collect standard system metrics. Configure Prometheus to scrape all nodes.

## Files to Create
- infrastructure/node-exporter/docker-compose.yml
- infrastructure/prometheus/scrape-configs/nodes.yml

## Dependencies
Task 024-001