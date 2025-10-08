# Task: Anthropic Provider Adapter (4-5 hours)
**ADR:** ADR-014  
**Date:** 2025-10-01

## Inputs
- Anthropic SDK
- Unified interface
- API credentials

## Outputs
- Anthropic adapter
- Response normalization
- Error handling

## Acceptance Criteria
- [ ] Implements LLMProvider interface
- [ ] Supports Claude 3.5 Sonnet, Claude 3 Haiku
- [ ] Normalizes Anthropic responses to OpenAI format
- [ ] Handles system message separation correctly
- [ ] Error mapping for Anthropic-specific errors
- [ ] Health check implementation
- [ ] Unit tests

## Implementation Details
Wrap Anthropic SDK. Convert between Anthropic message format (separate system) and unified format. Map Anthropic responses to match OpenAI structure for consistency.

## Files to Create
- packages/llm-gateway/src/providers/anthropic-adapter.ts
- packages/llm-gateway/src/providers/anthropic-normalizer.ts
- packages/llm-gateway/test/providers/anthropic.spec.ts

## Dependencies
Task 014-001