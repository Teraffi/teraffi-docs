# Task: Partnership Brief Matching (4-6 hours)
**ADR:** ADR-012  
**Date:** 2025-10-01

## Inputs
- Partnership brief text
- Pinecone multi-namespace search

## Outputs
- Brief matching service
- Cross-entity search (brands, content, creators)
- Relevance scoring

## Acceptance Criteria
- [ ] Embed partnership brief descriptions
- [ ] Query multiple namespaces (brand, content)
- [ ] Combine and rank results by relevance
- [ ] Consider feasibility factors (budget, timeline)
- [ ] Return top matches with explanations
- [ ] Integration tests with sample briefs

## Implementation Details
Accept partnership brief, construct embedding query, search across brand and content namespaces. Rank by semantic similarity plus business constraints (budget fit, timeline alignment).

## Files to Create
- packages/partnership-service/src/brief-matching.ts
- packages/partnership-service/src/relevance-scorer.ts
- packages/api/src/routes/partnership-matching.ts
- packages/partnership-service/test/brief-matching.spec.ts

## Dependencies
Task 012-004