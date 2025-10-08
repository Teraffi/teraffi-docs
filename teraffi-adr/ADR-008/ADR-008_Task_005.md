# Task: Capture Failure Matrix & Tests (3-5 hours)
**ADR:** ADR-008  
**Date:** 2025-10-01

## Inputs
- Payment error classes

## Outputs
- Matrix + tests

## Acceptance Criteria
- [ ] Transient→retry
- [ ] Permanent→cancel
- [ ] Ambiguous→hold

## Implementation Details
- Implement table-driven tests for matrix.

## Files to Create
- packages/worker/test/capture.matrix.spec.ts

## Dependencies
Task 008-002
