# Task: Observability: OpenTelemetry + App Insights Export (3-4 hours)
**ADR:** ADR-001  
**Date:** 2025-10-01

## Inputs
- OTel SDK
- App Insights key

## Outputs
- Tracing init
- Fastify instrumentation

## Acceptance Criteria
- [ ] Trace IDs in logs
- [ ] Spans for requests
- [ ] Exporter sends data

## Implementation Details
- Initialize OTel; instrument Fastify; add resource attrs (tenant, route).

## Files to Create
- packages/ops/src/otel.ts
- packages/api/src/instrumentation.ts

## Dependencies
Task 001-001
