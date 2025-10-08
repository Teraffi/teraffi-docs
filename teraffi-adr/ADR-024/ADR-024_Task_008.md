# Task: Tempo Deployment & Distributed Tracing (5-6 hours)
**ADR:** ADR-024  
**Date:** 2025-10-01

## Inputs
- Tempo requirements
- Trace retention policies

## Outputs
- Tempo server deployed
- Trace ingestion configured
- Grafana integration

## Acceptance Criteria
- [ ] Deploy Tempo on Azure
- [ ] Configure OTLP receiver
- [ ] Set trace retention (30 days)
- [ ] Add Tempo data source to Grafana
- [ ] Test trace queries
- [ ] Configure sampling (head-based)
- [ ] Set up object storage (Azure Blob)
- [ ] Documentation

## Implementation Details
Deploy Tempo for distributed tracing. Configure to receive traces from OpenTelemetry. Set appropriate retention and sampling. Connect to Grafana for trace visualization.

## Files to Create
- infrastructure/tempo/docker-compose.yml
- infrastructure/tempo/tempo-config.yml
- docs/setup/tempo_deployment.md

## Dependencies
Task 024-002, Task 024-003