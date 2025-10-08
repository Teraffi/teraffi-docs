# Task: Proposal Generation Service (6-8 hours)
**ADR:** ADR-017  
**Date:** 2025-10-01

## Inputs
- Deal details and structure
- Affinity data from Neo4j
- LLM Gateway

## Outputs
- Proposal generator
- Context gathering from graph
- Document storage

## Acceptance Criteria
- [ ] Gather context from Neo4j (affinities, trends, demographics)
- [ ] Generate proposal using GPT-4o
- [ ] Format as professional document (markdown)
- [ ] Store in Azure Blob Storage
- [ ] Record document metadata in database
- [ ] Link proposal to partnership record
- [ ] Processing time <2 minutes
- [ ] Unit tests with sample deals

## Implementation Details
Query Neo4j for shared affinities between entities. Use LLM to generate compelling proposal incorporating cultural alignment data. Store document in blob storage and record reference in database.

## Files to Create
- packages/deal-flow/src/proposal-generator.ts
- packages/deal-flow/src/context-gatherer.ts
- packages/deal-flow/test/proposal-generator.spec.ts

## Dependencies
Task 017-001, ADR-011 (Neo4j), ADR-014 (LLM Gateway)