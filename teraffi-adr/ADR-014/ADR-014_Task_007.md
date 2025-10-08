# Task: Provider Router & Failover (5-6 hours)
**ADR:** ADR-014  
**Date:** 2025-10-01

## Inputs
- Provider adapters
- Routing strategies
- Health check results

## Outputs
- Provider router service
- Routing strategy configuration
- Failover logic
- Health checking

## Acceptance Criteria
- [ ] Route requests based on model
- [ ] Define routing strategies per model
- [ ] Automatic failover to secondary provider
- [ ] Health check for provider availability
- [ ] Track failover events
- [ ] Configurable routing rules
- [ ] Unit tests for routing logic

## Implementation Details
Map models to preferred providers with fallback chains. Check provider health before routing. Implement failover logic when primary unavailable. Track and alert on failover events.

## Files to Create
- packages/llm-gateway/src/router.ts
- packages/llm-gateway/src/routing-strategies.ts
- packages/llm-gateway/src/health-checker.ts
- packages/llm-gateway/test/router.spec.ts

## Dependencies
Tasks 014-002, 014-003, 014-004