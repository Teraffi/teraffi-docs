# Task: GraphRAG Orchestration Service (4-6 hours)
**ADR:** ADR-013  
**Date:** 2025-10-01

## Inputs
- All component services
- Redis for caching
- PostgreSQL for logging

## Outputs
- Main GraphRAG service
- Query orchestration pipeline
- Caching layer

## Acceptance Criteria
- [ ] Orchestrate all 5 pipeline steps
- [ ] Multi-level caching (query, graph, semantic)
- [ ] Log queries to PostgreSQL
- [ ] Track performance metrics
- [ ] Handle errors at each step
- [ ] Return GraphRAGAnswer
- [ ] End-to-end integration tests

## Implementation Details
Wire together query understanding → graph retrieval → semantic augmentation → context assembly → answer generation. Implement caching at each level. Add error handling and fallbacks.

## Files to Create
- packages/graphrag-service/src/graphrag-service.ts
- packages/graphrag-service/src/cache-manager.ts
- packages/graphrag-service/src/query-logger.ts
- packages/graphrag-service/test/integration.spec.ts
- db/migrations/XXX_create_graphrag_queries.sql

## Dependencies
Tasks 013-001 through 013-005