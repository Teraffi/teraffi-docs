# Task: Context Assembly & Formatting (5-6 hours)
**ADR:** ADR-013  
**Date:** 2025-10-01

## Inputs
- GraphContext from Neo4j
- SemanticContext from Pinecone
- Original query

## Outputs
- Context assembly service
- Structured prompt for LLM
- Source tracking for citations

## Acceptance Criteria
- [ ] Format nodes, relationships, semantic matches
- [ ] Create citation IDs ([node_0], [rel_1], etc.)
- [ ] Assemble into structured prompt
- [ ] Estimate token count
- [ ] Truncate context if exceeds limit (8000 tokens)
- [ ] Prioritize high-value sources
- [ ] Unit tests for formatting

## Implementation Details
Transform graph and semantic data into natural language descriptions. Add citation markers. Assemble structured prompt with sections for entities, relationships, semantics. Track sources for citation extraction.

## Files to Create
- packages/graphrag-service/src/context-assembly.ts
- packages/graphrag-service/src/formatters.ts
- packages/graphrag-service/src/context-truncation.ts
- packages/graphrag-service/test/context-assembly.spec.ts

## Dependencies
Task 013-002, Task 013-003