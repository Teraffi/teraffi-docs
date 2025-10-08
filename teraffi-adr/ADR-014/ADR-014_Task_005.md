# Task: Response Cache Implementation (4-5 hours)
**ADR:** ADR-014  
**Date:** 2025-10-01

## Inputs
- Redis connection
- Request types
- Cache key generation strategy

## Outputs
- Response cache service
- Cache key generation
- TTL management

## Acceptance Criteria
- [ ] Generate stable cache keys from requests
- [ ] Store/retrieve completions and embeddings
- [ ] Configurable TTL (1h completions, 24h embeddings)
- [ ] Cache invalidation by pattern
- [ ] Cache statistics (hit rate, size)
- [ ] Unit tests with mocked Redis
- [ ] Integration tests with real Redis

## Implementation Details
Hash request parameters to generate cache keys. Store responses in Redis with appropriate TTL. Track cache hit/miss rates. Provide admin functions for cache management.

## Files to Create
- packages/llm-gateway/src/cache.ts
- packages/llm-gateway/src/cache-key-generator.ts
- packages/llm-gateway/test/cache.spec.ts

## Dependencies
Task 014-001, ADR-009 (Redis)