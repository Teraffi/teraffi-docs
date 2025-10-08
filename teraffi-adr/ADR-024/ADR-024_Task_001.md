# Task: Prometheus Deployment & Configuration (4-5 hours)
**ADR:** ADR-024  
**Date:** 2025-10-01

## Inputs
- Azure infrastructure
- Prometheus requirements

## Outputs
- Prometheus server deployed
- Configuration files
- Data retention policies
- Service discovery

## Acceptance Criteria
- [ ] Deploy Prometheus on Azure (3-node HA cluster)
- [ ] Configure scrape intervals (15s)
- [ ] Set up data retention (90 days)
- [ ] Configure service discovery
- [ ] Set up persistent storage
- [ ] Enable remote write (optional)
- [ ] Health checks passing
- [ ] Documentation

## Implementation Details
Deploy Prometheus cluster with high availability. Configure scrape targets for all services. Set appropriate retention. Configure persistent storage for time-series data.

## Files to Create
- infrastructure/prometheus/docker-compose.yml (or Kubernetes manifests)
- infrastructure/prometheus/prometheus.yml
- infrastructure/prometheus/alerts/
- docs/setup/prometheus_deployment.md

## Dependencies
Azure infrastructure