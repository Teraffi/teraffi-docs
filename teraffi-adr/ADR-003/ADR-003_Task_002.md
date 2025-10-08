# Task: Relay (Phase 1: Polling) + Metrics (4-6 hours)
**ADR:** ADR-003  
**Date:** 2025-10-01

## Inputs
- Repo
- Service Bus

## Outputs
- Relay loop + metrics

## Acceptance Criteria
- [ ] Publishes batches
- [ ] Updates published_at
- [ ] Lag/EPS gauges

## Implementation Details
- Implement polling loop; publish with correlation IDs; emit metrics.

## Files to Create
- packages/ops/src/outboxRelay.ts
- packages/ops/src/outboxMetrics.ts

## Dependencies
Task 003-001
