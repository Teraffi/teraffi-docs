# Task: API Error Envelope & Mapper (2-3 hours)
**ADR:** ADR-001  
**Date:** 2025-10-01

## Inputs
- Error taxonomy
- AJV errors

## Outputs
- Error formatter util

## Acceptance Criteria
- [ ] Consistent JSON error
- [ ] AJV errors mapped
- [ ] Unit tests

## Implementation Details
- Implement error mapper and register as Fastify error handler.

## Files to Create
- packages/api/src/lib/error.ts
- packages/api/test/error.spec.ts

## Dependencies
Task 001-002
