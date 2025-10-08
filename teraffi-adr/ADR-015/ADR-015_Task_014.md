# Task: Comprehensive Test Suite (5-7 hours)
**ADR:** ADR-015  
**Date:** 2025-10-01

## Inputs
- All affinity engine components
- Test fixtures and sample data

## Outputs
- Unit tests
- Integration tests
- Performance tests
- End-to-end tests

## Acceptance Criteria
- [ ] Triple extraction tests (various inputs)
- [ ] Public data aggregation tests
- [ ] Component scoring tests
- [ ] Full affinity calculation tests
- [ ] Cache behavior tests
- [ ] Performance benchmarks (20-30s first, 1-2s cached)
- [ ] Goal extraction accuracy tests
- [ ] CI green

## Implementation Details
Create comprehensive test suite covering all components. Use fixtures for consistent inputs. Mock external services (LLM, public APIs) for unit tests. Real integration tests in staging.

## Files to Create
- packages/affinity-engine/test/e2e.spec.ts
- packages/affinity-engine/test/performance.spec.ts
- packages/affinity-engine/test/fixtures/entities.json
- packages/affinity-engine/test/fixtures/goals.json

## Dependencies
Tasks 015-001 through 015-013