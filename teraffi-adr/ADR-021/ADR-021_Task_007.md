# Task: Graph-Based Search (4-5 hours)
**ADR:** ADR-021  
**Date:** 2025-10-01

## Inputs
- Neo4j graph database
- User's network
- Search query

## Outputs
- Graph search service
- Network-aware queries
- Distance-based scoring

## Acceptance Criteria
- [ ] Search entities in user's network (1-2 hops)
- [ ] Match on name, values, trends
- [ ] Score by graph distance
- [ ] Return connected entities
- [ ] Combine with other search signals
- [ ] Performance <200ms
- [ ] Unit tests

## Implementation Details
Use Neo4j to find entities connected to user's network. Score by graph distance (closer = higher score). Useful for "similar to my partners" queries.

## Files to Create
- packages/search/src/graph-search.ts
- packages/search/test/graph-search.spec.ts

## Dependencies
Task 021-004, ADR-011 (Neo4j)