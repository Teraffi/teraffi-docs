# Task: Twitter/X API Integration & Signal Collection (5-7 hours)
**ADR:** ADR-016  
**Date:** 2025-10-01

## Inputs
- Twitter API v2 credentials
- Queue infrastructure (BullMQ)
- Signal schema definitions

## Outputs
- Twitter collector service
- Streaming and search capabilities
- Signal normalization

## Acceptance Criteria
- [ ] Authenticate with Twitter API v2
- [ ] Implement streaming endpoint for real-time signals
- [ ] Implement search endpoint for keyword tracking
- [ ] Extract engagement metrics (likes, retweets, replies)
- [ ] Normalize to SocialSignal format
- [ ] Enqueue signals for processing
- [ ] Handle rate limits gracefully
- [ ] Unit tests with mocked Twitter client

## Implementation Details
Use Twitter API v2 to collect signals via streaming and search endpoints. Extract text, engagement metrics, hashtags, mentions. Normalize to common SocialSignal interface. Handle rate limits with exponential backoff.

## Files to Create
- packages/social-listening/src/collectors/twitter-collector.ts
- packages/social-listening/src/types/social-signal.ts
- packages/social-listening/test/twitter-collector.spec.ts
- config/twitter-api.ts

## Dependencies
ADR-003 (Queue infrastructure)