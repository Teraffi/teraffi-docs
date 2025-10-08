# Task: Entity Embedding Sync (Brand, Member, Content) (5-7 hours)
**ADR:** ADR-012  
**Date:** 2025-10-01

## Inputs
- PostgreSQL entity data
- Outbox events
- Embedding pipeline

## Outputs
- Event handlers for entity changes
- Text construction logic
- Pinecone upsert operations

## Acceptance Criteria
- [ ] Handle brand.created, brand.values_updated events
- [ ] Handle member.profile_updated events
- [ ] Handle content.created, content.updated events
- [ ] Construct rich text representations for embedding
- [ ] Upsert to correct Pinecone namespace
- [ ] Update embedding_status table in PostgreSQL
- [ ] Calculate text hash for change detection

## Implementation Details
Subscribe to entity change events. Construct rich text from entity attributes (name, values, mission, etc.). Enqueue embedding job. After embedding generated, upsert to Pinecone with metadata.

## Files to Create
- packages/embedding-service/src/handlers/brand.ts
- packages/embedding-service/src/handlers/member.ts
- packages/embedding-service/src/handlers/content.ts
- packages/embedding-service/src/text-builder.ts
- db/migrations/XXX_create_embedding_status.sql

## Dependencies
Task 012-002, ADR-003 (Outbox events)