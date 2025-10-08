# Task: Graph Algorithm Integration (PageRank, Community Detection) (5-7 hours)
**ADR:** ADR-011  
**Date:** 2025-10-01

## Inputs
- Neo4j Graph Data Science library
- Member/Brand graph

## Outputs
- Influence scoring (PageRank)
- Community detection (Louvain)
- Integration with affinity service

## Acceptance Criteria
- [ ] PageRank algorithm configured and tested
- [ ] Louvain community detection working
- [ ] Results persisted as node properties (nightly batch)
- [ ] Influence scores exposed via API
- [ ] Algorithm performance acceptable (<5min for 100k nodes)

## Implementation Details
Use Neo4j GDS library for graph algorithms. Project Member/Brand subgraph. Run PageRank to calculate influence scores. Store results on nodes. Schedule nightly recalculation.

## Files to Create
- packages/graph-algorithms/src/pagerank.ts
- packages/graph-algorithms/src/louvain.ts
- packages/graph-algorithms/src/batch-job.ts
- db/neo4j/migrations/003_algorithm_properties.cypher

## Dependencies
Task 011-003 (sufficient data), Task 011-004