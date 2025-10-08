# Task: Prometheus Metrics Endpoint (3-4 hours)
**ADR:** ADR-024  
**Date:** 2025-10-01

## Inputs
- Prometheus exporter
- HTTP server
- Default metrics

## Outputs
- /metrics endpoint
- Default metric collection
- HTTP middleware

## Acceptance Criteria
- [ ] Expose /metrics endpoint on port 9464
- [ ] Collect default metrics (CPU, memory, GC)
- [ ] Instrument HTTP requests (duration, count, errors)
- [ ] Track active connections
- [ ] Prometheus scrape-compatible format
- [ ] No authentication on /metrics (internal network only)
- [ ] Unit tests

## Implementation Details
Add /metrics endpoint to each service. Expose Prometheus-formatted metrics. Collect default Node.js metrics. Instrument HTTP middleware to track requests.

## Files to Create
- packages/api/src/middleware/metrics.ts
- packages/core/src/telemetry/prometheus-exporter.ts
- packages/api/test/metrics.spec.ts

## Dependencies
Task 024-004