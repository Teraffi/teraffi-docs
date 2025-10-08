# Task: OpenTelemetry SDK Setup (5-6 hours)
**ADR:** ADR-024  
**Date:** 2025-10-01

## Inputs
- OpenTelemetry SDK
- Service metadata
- Exporter configurations

## Outputs
- OpenTelemetry initialization
- Auto-instrumentation
- Manual instrumentation helpers

## Acceptance Criteria
- [ ] Initialize OpenTelemetry SDK
- [ ] Configure service metadata (name, version, environment)
- [ ] Set up auto-instrumentation (HTTP, PostgreSQL, Redis)
- [ ] Configure metrics exporter (Prometheus)
- [ ] Configure trace exporter (Tempo)
- [ ] Configure log exporter (Loki)
- [ ] Graceful shutdown handling
- [ ] Unit tests

## Implementation Details
Set up OpenTelemetry SDK in each service. Configure auto-instrumentation for common libraries. Export metrics, traces, and logs to appropriate backends.

## Files to Create
- packages/core/src/telemetry/otel-setup.ts
- packages/core/src/telemetry/config.ts
- packages/core/test/telemetry.spec.ts

## Dependencies
Task 024-001