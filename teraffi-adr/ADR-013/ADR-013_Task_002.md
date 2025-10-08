# Task: Graph Retrieval & Cypher Generation (6-8 hours)
**ADR:** ADR-013  
**Date:** 2025-10-01

## Inputs
- QueryIntent
- Neo4j connection
- Query pattern templates

## Outputs
- Graph retrieval service
- Dynamic Cypher query builder
- Subgraph extraction

## Acceptance Criteria
- [ ] Build Cypher queries from intent
- [ ] Support common patterns (find, recommend, analyze)
- [ ] Limit traversal depth (2-3 hops max)
- [ ] Extract nodes and relationships from results
- [ ] Return formatted GraphContext
- [ ] p95 latency <2 seconds
- [ ] Handle tenant filtering

## Implementation Details
Map intent to Cypher query templates. Build parameterized queries based on entity types and constraints. Execute and parse Neo4j results into GraphContext structure.

## Files to Create
- packages/graphrag-service/src/graph-retrieval.ts
- packages/graphrag-service/src/cypher-builder.ts
- packages/graphrag-service/src/types/graph-context.ts
- packages/graphrag-service/test/graph-retrieval.spec.ts

## Dependencies
Task 013-001, ADR-011 (Neo4j)