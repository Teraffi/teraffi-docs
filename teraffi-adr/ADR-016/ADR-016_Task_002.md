# Task: Reddit API Integration (4-5 hours)
**ADR:** ADR-016  
**Date:** 2025-10-01

## Inputs
- Reddit API credentials (Snoowrap)
- Queue infrastructure
- Subreddit list

## Outputs
- Reddit collector service
- Subreddit monitoring
- Trending post collection

## Acceptance Criteria
- [ ] Authenticate with Reddit API
- [ ] Monitor specified subreddits
- [ ] Collect hot/trending posts
- [ ] Extract engagement (upvotes, comments)
- [ ] Normalize to SocialSignal format
- [ ] Handle Reddit rate limits
- [ ] Unit tests

## Implementation Details
Use Snoowrap library to access Reddit API. Monitor configurable list of subreddits. Collect hot posts and normalize to SocialSignal format. Handle rate limits (60 requests/minute).

## Files to Create
- packages/social-listening/src/collectors/reddit-collector.ts
- packages/social-listening/test/reddit-collector.spec.ts
- config/reddit-api.ts

## Dependencies
Task 016-001 (SocialSignal types)