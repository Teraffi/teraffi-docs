# Task: Custom Metrics Implementation (5-6 hours)
**ADR:** ADR-024  
**Date:** 2025-10-01

## Inputs
- Business metric requirements
- Application events

## Outputs
- Custom metric definitions
- Metric recording helpers
- Business KPI tracking

## Acceptance Criteria
- [ ] Define custom counters (partnerships created, searches, etc.)
- [ ] Define custom histograms (latency, duration)
- [ ] Define custom gauges (active users, queue depth)
- [ ] Create helper functions for recording
- [ ] Add labels for dimensionality
- [ ] Document all custom metrics
- [ ] Unit tests

## Implementation Details
Define application-specific metrics using OpenTelemetry API. Create counters for business events, histograms for latency, gauges for state. Provide easy-to-use recording functions.

## Files to Create
- packages/core/src/telemetry/metrics.ts
- packages/core/src/telemetry/business-metrics.ts
- docs/metrics/custom_metrics_catalog.md

## Dependencies
Task 024-003