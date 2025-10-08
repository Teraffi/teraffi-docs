# Task: News Aggregation Service (4-5 hours)
**ADR:** ADR-016  
**Date:** 2025-10-01

## Inputs
- RSS feed URLs
- News API credentials
- Queue infrastructure

## Outputs
- News aggregator service
- RSS feed parser
- Article normalization

## Acceptance Criteria
- [ ] Parse RSS feeds from news sources
- [ ] Query News API for topics
- [ ] Extract article text, publish date, source
- [ ] Normalize to SocialSignal format
- [ ] Deduplicate articles
- [ ] Schedule periodic collection (hourly)
- [ ] Unit tests

## Implementation Details
Use RSS parser for feeds, News API for additional sources. Extract article content and metadata. Normalize to SocialSignal format with source='news'. Schedule collection via cron job.

## Files to Create
- packages/social-listening/src/collectors/news-collector.ts
- packages/social-listening/src/parsers/rss-parser.ts
- packages/social-listening/test/news-collector.spec.ts

## Dependencies
Task 016-001