# Task: Entity Index Design & Mapping (3-4 hours)
**ADR:** ADR-021  
**Date:** 2025-10-01

## Inputs
- Entity schema requirements
- Search field specifications
- Vector dimension requirements

## Outputs
- Index mappings
- Field configurations
- Analyzer definitions

## Acceptance Criteria
- [ ] Define entities index mapping
- [ ] Configure text fields with analyzers
- [ ] Configure keyword fields for facets
- [ ] Configure geo_point for location search
- [ ] Configure dense_vector for embeddings (1536 dims)
- [ ] Set up multi-fields (text + keyword + ngram)
- [ ] Configure aggregations
- [ ] Create index with mapping
- [ ] Documentation

## Implementation Details
Design Elasticsearch index mapping for entities. Configure appropriate field types and analyzers. Set up multi-fields for different query types. Configure dense_vector for semantic search.

## Files to Create
- packages/search/src/mappings/entities-mapping.json
- packages/search/src/mappings/trends-mapping.json
- docs/search/index_design.md

## Dependencies
Task 021-001