# Task: Query Understanding & Intent Extraction (5-7 hours)
**ADR:** ADR-013  
**Date:** 2025-10-01

## Inputs
- LLM Gateway client
- Natural language queries
- Intent schema definitions

## Outputs
- Query understanding service
- Intent extraction with structured output
- Constraint parsing

## Acceptance Criteria
- [ ] Parse natural language to structured QueryIntent
- [ ] Extract entity types, actions, constraints
- [ ] Reformulate semantic query for embedding
- [ ] Use GPT-4o-mini for cost efficiency
- [ ] JSON output validation
- [ ] p95 latency <1 second
- [ ] Unit tests with diverse query examples

## Implementation Details
Use LLM to parse queries into structured intent (primary_action, entity_types, constraints, semantic_query). Implement prompt engineering for consistent JSON output. Validate and normalize extracted constraints.

## Files to Create
- packages/graphrag-service/src/query-understanding.ts
- packages/graphrag-service/src/types/intent.ts
- packages/graphrag-service/test/query-understanding.spec.ts
- packages/graphrag-service/test/fixtures/sample-queries.json

## Dependencies
ADR-014 (LLM Gateway)