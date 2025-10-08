# Task: OpenAI Provider Adapter (4-5 hours)
**ADR:** ADR-014  
**Date:** 2025-10-01

## Inputs
- OpenAI SDK
- Unified interface from Task 001
- API credentials from Key Vault

## Outputs
- OpenAI adapter implementation
- Completion and embedding support
- Error handling

## Acceptance Criteria
- [ ] Implements LLMProvider interface
- [ ] Supports GPT-4o, GPT-4o-mini models
- [ ] Supports text-embedding-3-large
- [ ] Proper error mapping (rate limits, timeouts, etc.)
- [ ] Health check endpoint
- [ ] Unit tests with mocked OpenAI client
- [ ] Retry logic with exponential backoff

## Implementation Details
Wrap OpenAI SDK to implement unified interface. Map OpenAI-specific responses to standard format. Handle rate limits, timeouts, and API errors gracefully.

## Files to Create
- packages/llm-gateway/src/providers/openai-adapter.ts
- packages/llm-gateway/src/providers/openai-error-mapper.ts
- packages/llm-gateway/test/providers/openai.spec.ts

## Dependencies
Task 014-001