# Task: Embedding Generation Pipeline (6-8 hours)
**ADR:** ADR-012  
**Date:** 2025-10-01

## Inputs
- LLM Gateway client
- Entity data from PostgreSQL
- Queue infrastructure (BullMQ)

## Outputs
- Embedding generation service
- Queue workers for async processing
- Batch embedding support

## Acceptance Criteria
- [ ] Generate embeddings via LLM Gateway (text-embedding-3-large)
- [ ] Process embedding jobs from queue
- [ ] Batch processing (up to 100 texts at once)
- [ ] Retry logic with exponential backoff
- [ ] Idempotent (duplicate jobs safe)
- [ ] Throughput >1000 embeddings/minute

## Implementation Details
Implement EmbeddingPipeline class with queue-based processing. Use LLM Gateway to call OpenAI embedding API. Support batch requests for efficiency. Handle rate limits gracefully.

## Files to Create
- packages/embedding-service/src/pipeline.ts
- packages/embedding-service/src/batch-processor.ts
- packages/embedding-service/src/queue-worker.ts
- packages/embedding-service/test/pipeline.spec.ts

## Dependencies
Task 012-001, ADR-014 (LLM Gateway), ADR-003 (Queue infrastructure)