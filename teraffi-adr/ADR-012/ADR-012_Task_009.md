# Task: Comprehensive Embedding & Search Test Suite (5-7 hours)
**ADR:** ADR-012  
**Date:** 2025-10-01

## Inputs
- All embedding services
- Test data fixtures

## Outputs
- Unit tests
- Integration tests
- Performance benchmarks

## Acceptance Criteria
- [ ] Embedding generation tests (including batching)
- [ ] Pinecone upsert/query tests
- [ ] Semantic search correctness tests
- [ ] Cache behavior tests
- [ ] Performance benchmarks (latency, throughput)
- [ ] Error handling tests (API failures, timeouts)
- [ ] CI green

## Implementation Details
Create comprehensive test suite covering embedding pipeline, search queries, caching, and error scenarios. Use test containers for Pinecone emulation if available, otherwise mock.

## Files to Create
- packages/embedding-service/test/integration.spec.ts
- packages/search-service/test/semantic-search.spec.ts
- packages/search-service/test/performance.spec.ts
- packages/embedding-service/test/error-handling.spec.ts

## Dependencies
Tasks 012-001 through 012-008