# Task: Conversational Context Management (4-5 hours)
**ADR:** ADR-013  
**Date:** 2025-10-01

## Inputs
- Redis for conversation storage
- GraphRAG query results
- Message history

## Outputs
- Conversation manager
- Follow-up question handling
- Context reuse

## Acceptance Criteria
- [ ] Create and track conversations
- [ ] Store message history in Redis (1 hour TTL)
- [ ] Reuse graph context for follow-ups
- [ ] Format conversation history for LLM
- [ ] Handle conversation expiry
- [ ] Reduce costs on follow-up queries
- [ ] Integration tests

## Implementation Details
Store conversation context in Redis with conversation_id. When follow-up detected, append to existing conversation and reuse previously retrieved graph context. Include message history in LLM prompt.

## Files to Create
- packages/graphrag-service/src/conversation-manager.ts
- packages/graphrag-service/src/types/conversation.ts
- packages/graphrag-service/test/conversation.spec.ts

## Dependencies
Task 013-006, ADR-009 (Redis)