# Task: Service Instrumentation (All Services) (8-10 hours)
**ADR:** ADR-024  
**Date:** 2025-10-01

## Inputs
- OpenTelemetry SDK
- All application services

## Outputs
- Instrumented services
- Metrics exposed
- Traces generated

## Acceptance Criteria
- [ ] Instrument API service
- [ ] Instrument Deal Flow service
- [ ] Instrument Search service
- [ ] Instrument Billing service
- [ ] Instrument Notification service
- [ ] All services expose /metrics
- [ ] All services generate traces
- [ ] All services use structured logging
- [ ] Integration tests

## Implementation Details
Add OpenTelemetry instrumentation to all services. Expose metrics, generate traces, use structured logging. Ensure consistent implementation across services.

## Files to Create
- packages/api/src/index.ts (add telemetry init)
- packages/deal-flow/src/index.ts (add telemetry init)
- packages/search/src/index.ts (add telemetry init)
- (repeat for all services)

## Dependencies
Task 024-003, Task 024-004, Task 024-006