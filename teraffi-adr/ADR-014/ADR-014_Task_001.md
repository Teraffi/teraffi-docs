# Task: Core Gateway Interface & Types (3-4 hours)
**ADR:** ADR-014  
**Date:** 2025-10-01

## Inputs
- API specifications from OpenAI, Anthropic, Ollama
- Common interface requirements

## Outputs
- Unified TypeScript interfaces
- Request/response types
- Provider interface contract

## Acceptance Criteria
- [ ] CompletionRequest/Response types defined
- [ ] EmbeddingRequest/Response types defined
- [ ] LLMProvider interface defined
- [ ] Message, Usage types defined
- [ ] Type safety enforced across all providers
- [ ] Documentation with examples

## Implementation Details
Define unified interfaces that abstract differences between OpenAI, Anthropic, and Ollama. Create type definitions that all adapters must implement. Ensure consistent error handling types.

## Files to Create
- packages/llm-gateway/src/types.ts
- packages/llm-gateway/src/interfaces/provider.ts
- packages/llm-gateway/src/types/index.ts

## Dependencies
None (foundation task)