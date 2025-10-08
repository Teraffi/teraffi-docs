# Task: Natural Language Input Processing & Triple Extraction (6-8 hours)
**ADR:** ADR-015  
**Date:** 2025-10-01

## Inputs
- LLM Gateway client
- Two-question user input ("Who are you?" + "What do you want?")
- Triple schema definitions

## Outputs
- Triple extraction service
- Structured triple output (facts, people, places, things, brands, emotions, colors, genres, demographics)
- Confidence scoring

## Acceptance Criteria
- [ ] Parse natural language using LLM (GPT-4o)
- [ ] Extract semantic triples in [subject, predicate, object] format
- [ ] Categorize triples into 9 types
- [ ] Confidence scoring for each triple
- [ ] Handle both questions in single processing pass
- [ ] Processing time <15 seconds for extraction
- [ ] Unit tests with sample inputs

## Implementation Details
Use LLM Gateway to parse two-question input into structured semantic triples. Implement prompt engineering for consistent JSON output across all triple categories. Validate triple structure and confidence thresholds.

## Files to Create
- packages/affinity-engine/src/triple-extraction.ts
- packages/affinity-engine/src/types/triples.ts
- packages/affinity-engine/test/triple-extraction.spec.ts
- packages/affinity-engine/test/fixtures/sample-inputs.json

## Dependencies
ADR-014 (LLM Gateway)