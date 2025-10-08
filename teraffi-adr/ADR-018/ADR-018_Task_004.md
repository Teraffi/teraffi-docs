# Task: Neo4j to Staging Extract (4-5 hours)
**ADR:** ADR-018  
**Date:** 2025-10-01

## Inputs
- Neo4j graph database
- Affinity and trend data
- Extract requirements

## Outputs
- Extract scripts for graph data
- Relationship flattening
- Staging storage

## Acceptance Criteria
- [ ] Extract PARTNERS_WITH relationships
- [ ] Extract ALIGNS_WITH relationships
- [ ] Extract Trend nodes with momentum metrics
- [ ] Extract APPEALS_TO demographic relationships
- [ ] Flatten graph structure to tabular format
- [ ] Handle graph traversals efficiently
- [ ] Store in staging area
- [ ] Error handling

## Implementation Details
Query Neo4j for relevant nodes and relationships. Flatten graph structure into tabular format suitable for warehouse. Export affinity data, trend metrics, and partnership relationships.

## Files to Create
- analytics/etl/extract/neo4j_extract.py
- analytics/etl/extract/graph_flattener.py
- analytics/etl/test/test_neo4j_extract.py

## Dependencies
Task 018-002, ADR-011 (Neo4j)