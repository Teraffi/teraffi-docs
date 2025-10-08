# Task: Knowledge Graph Population (5-7 hours)
**ADR:** ADR-015  
**Date:** 2025-10-01

## Inputs
- Extracted triples (user + public data)
- Neo4j connection
- User ID and timestamp

## Outputs
- Graph population service
- Node and relationship creation
- Predicate normalization

## Acceptance Criteria
- [ ] Create nodes from triple subjects and objects
- [ ] Create relationships with normalized predicates
- [ ] Store confidence scores and sources
- [ ] Track timestamps for freshness
- [ ] Handle duplicate entities (MERGE not CREATE)
- [ ] Batch operations for performance
- [ ] Trigger embedding generation for new entities
- [ ] Integration tests with Neo4j

## Implementation Details
Transform semantic triples into Neo4j graph structure. Normalize predicates to consistent relationship types. Use MERGE for idempotent operations. Track data lineage (source, timestamp, confidence).

## Files to Create
- packages/affinity-engine/src/graph-population.ts
- packages/affinity-engine/src/predicate-normalizer.ts
- packages/affinity-engine/test/graph-population.spec.ts
- db/neo4j/migrations/005_affinity_triples.cypher

## Dependencies
Task 015-001, Task 015-002, ADR-011 (Neo4j)