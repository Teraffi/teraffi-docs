# Task: Ollama Provider Adapter (3-4 hours)
**ADR:** ADR-014  
**Date:** 2025-10-01

## Inputs
- Ollama HTTP API
- Unified interface
- Local Ollama instance

## Outputs
- Ollama adapter
- Completion and embedding support
- Response normalization

## Acceptance Criteria
- [ ] Implements LLMProvider interface
- [ ] Supports Llama 3.1, Mistral models
- [ ] Supports nomic-embed-text embeddings
- [ ] HTTP client for Ollama API
- [ ] Response normalization to unified format
- [ ] Configurable base URL
- [ ] Unit tests with mocked HTTP

## Implementation Details
Implement HTTP client for Ollama REST API. Normalize Ollama responses to match unified format. Handle local deployment specifics (longer timeouts, different error patterns).

## Files to Create
- packages/llm-gateway/src/providers/ollama-adapter.ts
- packages/llm-gateway/src/providers/ollama-client.ts
- packages/llm-gateway/test/providers/ollama.spec.ts

## Dependencies
Task 014-001