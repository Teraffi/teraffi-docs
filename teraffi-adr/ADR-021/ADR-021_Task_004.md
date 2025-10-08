# Task: Search Indexer Service (5-6 hours)
**ADR:** ADR-021  
**Date:** 2025-10-01

## Inputs
- Entity data from PostgreSQL
- Graph data from Neo4j
- Embeddings

## Outputs
- Indexer service
- Bulk indexing
- Incremental updates
- Scheduled reindexing

## Acceptance Criteria
- [ ] Index single entity
- [ ] Bulk index entities (batches of 100)
- [ ] Enrich with graph data (partnerships, trends, demographics)
- [ ] Generate and store embeddings
- [ ] Handle updates (reindex on change)
- [ ] Handle deletes (remove from index)
- [ ] Full reindex capability
- [ ] Error handling and retries
- [ ] Unit tests

## Implementation Details
Build indexer that pulls data from PostgreSQL and Neo4j. Generate embeddings. Construct search documents. Bulk index into Elasticsearch. Support incremental updates.

## Files to Create
- packages/search/src/indexer.ts
- packages/search/src/enrichment/graph-enricher.ts
- packages/search/src/jobs/reindex-all.ts
- packages/search/test/indexer.spec.ts

## Dependencies
Tasks 021-002, 021-003, ADR-011 (Neo4j)