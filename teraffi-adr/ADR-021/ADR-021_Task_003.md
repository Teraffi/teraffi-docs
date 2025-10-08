# Task: Embedding Generation Service (4-5 hours)
**ADR:** ADR-021  
**Date:** 2025-10-01

## Inputs
- OpenAI API credentials
- Entity text content
- LLM Gateway

## Outputs
- Embedding service
- Batch processing
- Caching layer

## Acceptance Criteria
- [ ] Generate embeddings using OpenAI text-embedding-3-small
- [ ] Build rich text from entity fields
- [ ] Batch processing for efficiency
- [ ] Cache embeddings (Redis)
- [ ] Error handling and retries
- [ ] Cost tracking
- [ ] Processing time <500ms per entity
- [ ] Unit tests

## Implementation Details
Use OpenAI Embeddings API via LLM Gateway. Concatenate entity fields into rich text representation. Batch requests for cost efficiency. Cache to avoid regenerating.

## Files to Create
- packages/search/src/embedding-service.ts
- packages/search/src/text-builder.ts
- packages/search/test/embedding-service.spec.ts

## Dependencies
Task 021-002, ADR-014 (LLM Gateway)