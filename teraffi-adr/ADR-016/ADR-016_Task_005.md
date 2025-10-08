# Task: LLM Trend Extraction Service (5-7 hours)
**ADR:** ADR-016  
**Date:** 2025-10-01

## Inputs
- Filtered social signals
- LLM Gateway client
- Trend schema

## Outputs
- Trend extractor service
- Batch processing
- Trend deduplication

## Acceptance Criteria
- [ ] Batch signals (50 per request) for efficiency
- [ ] Use GPT-4o-mini for trend extraction
- [ ] Extract trend name, category, keywords, sentiment
- [ ] Identify demographic appeal
- [ ] Deduplicate and merge similar trends
- [ ] Confidence scoring
- [ ] Processing time <2 minutes per batch
- [ ] Unit tests with sample signals

## Implementation Details
Batch signals for LLM analysis. Use structured prompts to extract trends consistently. Deduplicate by trend name (case-insensitive). Merge keywords and metadata for duplicates.

## Files to Create
- packages/social-listening/src/trend-extraction.ts
- packages/social-listening/src/trend-deduplicator.ts
- packages/social-listening/test/trend-extraction.spec.ts

## Dependencies
Task 016-004, ADR-014 (LLM Gateway)