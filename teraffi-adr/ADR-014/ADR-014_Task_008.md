# Task: Main LLM Gateway Service (5-7 hours)
**ADR:** ADR-014  
**Date:** 2025-10-01

## Inputs
- All provider adapters
- Cache service
- Cost tracker
- Router

## Outputs
- Main LLM Gateway orchestration
- Complete/embed methods
- Retry logic
- Error handling

## Acceptance Criteria
- [ ] Orchestrate cache → route → execute → track
- [ ] Implement retry with exponential backoff
- [ ] Handle errors and fallbacks
- [ ] Return enriched responses (cached flag, provider, etc.)
- [ ] Log requests for audit
- [ ] Integration tests end-to-end
- [ ] Performance tests (latency targets)

## Implementation Details
Wire together all components: check cache, route to provider, execute with retry, track cost, cache response. Handle all error scenarios with appropriate fallbacks.

## Files to Create
- packages/llm-gateway/src/gateway.ts
- packages/llm-gateway/src/retry-handler.ts
- packages/llm-gateway/test/integration.spec.ts
- packages/llm-gateway/test/performance.spec.ts

## Dependencies
Tasks 014-002 through 014-007