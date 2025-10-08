# Task: Public Data Aggregation Service (8-10 hours)
**ADR:** ADR-015  
**Date:** 2025-10-01

## Inputs
- Entity names from triple extraction
- API credentials for data sources

## Outputs
- Public data aggregator
- Wikipedia, IMDb, social media, earnings report integrations
- Enriched triple generation

## Acceptance Criteria
- [ ] Query Wikipedia API for entity data
- [ ] Query IMDb API for entertainment entities
- [ ] Query social media APIs (Twitter, etc.)
- [ ] Query financial data sources (earnings reports)
- [ ] Extract additional triples from public data
- [ ] Rate limiting and quota management
- [ ] Caching to avoid duplicate queries
- [ ] Processing time <10 seconds for 5 entities

## Implementation Details
Build service to enrich user-provided entities with public data. Implement parallel querying of multiple sources. Extract semantic triples from unstructured public data. Handle API failures gracefully.

## Files to Create
- packages/affinity-engine/src/public-data-aggregator.ts
- packages/affinity-engine/src/sources/wikipedia.ts
- packages/affinity-engine/src/sources/imdb.ts
- packages/affinity-engine/src/sources/social-media.ts
- packages/affinity-engine/test/public-data.spec.ts

## Dependencies
Task 015-001