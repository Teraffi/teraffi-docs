# Task: Consumer Skeleton + Idempotency (3-4 hours)
**ADR:** ADR-003  
**Date:** 2025-10-01

## Inputs
- Service Bus client

## Outputs
- Consumer with dedupe

## Acceptance Criteria
- [ ] At-least-once safe
- [ ] Checkpoint stored
- [ ] Unit tests

## Implementation Details
- Implement dedupe/lastProcessed; idempotent handler interface.

## Files to Create
- packages/ops/src/consumer.ts
- packages/ops/test/consumer.spec.ts

## Dependencies
Task 003-001
