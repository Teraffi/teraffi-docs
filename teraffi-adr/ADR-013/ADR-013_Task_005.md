# Task: Answer Generation & Citation Extraction (5-7 hours)
**ADR:** ADR-013  
**Date:** 2025-10-01

## Inputs
- Assembled context
- LLM Gateway
- Original query

## Outputs
- Answer generation service
- Citation extraction
- Confidence scoring

## Acceptance Criteria
- [ ] Generate answers using GPT-4o
- [ ] Extract citations from answer text
- [ ] Map citations to source objects
- [ ] Calculate confidence score
- [ ] Extract reasoning path
- [ ] Include token usage metadata
- [ ] Handle LLM failures gracefully
- [ ] Unit tests with mock LLM responses

## Implementation Details
Submit structured context to LLM with system prompt emphasizing citations and factual answers. Parse response for citation markers. Calculate confidence from citation count and semantic scores.

## Files to Create
- packages/graphrag-service/src/answer-generation.ts
- packages/graphrag-service/src/citation-extractor.ts
- packages/graphrag-service/src/confidence-scorer.ts
- packages/graphrag-service/test/answer-generation.spec.ts

## Dependencies
Task 013-004, ADR-014 (LLM Gateway)