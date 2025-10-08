# Task: Query Intent Parser (4-5 hours)
**ADR:** ADR-021  
**Date:** 2025-10-01

## Inputs
- User search query
- LLM for intent detection

## Outputs
- Intent parser
- Keyword extraction
- Filter inference

## Acceptance Criteria
- [ ] Parse query with LLM (GPT-4o-mini)
- [ ] Extract keywords
- [ ] Detect intent (find_partners, discover_trends, etc.)
- [ ] Infer filters from natural language
- [ ] Handle ambiguous queries
- [ ] Fallback to keyword search if parsing fails
- [ ] Unit tests

## Implementation Details
Use LLM to understand query intent. Extract structured data (keywords, filters). For example, "Gen Z brands in fashion" → filters: {demographics: ['gen_z'], industry: ['fashion']}.

## Files to Create
- packages/search/src/intent-parser.ts
- packages/search/test/intent-parser.spec.ts

## Dependencies
Task 021-005, ADR-014 (LLM Gateway)