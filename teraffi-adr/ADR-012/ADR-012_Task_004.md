# Task: Semantic Search Service (5-7 hours)
**ADR:** ADR-012  
**Date:** 2025-10-01

## Inputs
- Pinecone index
- Query text
- Metadata filters

## Outputs
- Semantic search service
- REST API endpoints
- Query result caching

## Acceptance Criteria
- [ ] Generate query embeddings
- [ ] Query Pinecone with hybrid search (vector + metadata)
- [ ] Support filtering (industry, revenue, location, etc.)
- [ ] Return results with similarity scores
- [ ] Cache frequent queries in Redis (1 hour TTL)
- [ ] p95 latency <200ms
- [ ] Unit tests with mocked Pinecone

## Implementation Details
Accept search query text, generate embedding, query Pinecone with filters. Implement caching layer for common queries. Support hybrid search (semantic + exact match filters).

## Files to Create
- packages/search-service/src/semantic-search.ts
- packages/search-service/src/query-builder.ts
- packages/search-service/src/cache.ts
- packages/api/src/routes/search.ts
- packages/search-service/test/semantic-search.spec.ts

## Dependencies
Task 012-001, Task 012-003, ADR-009 (Redis)