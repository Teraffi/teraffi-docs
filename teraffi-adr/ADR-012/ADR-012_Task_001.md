# Task: Pinecone Account Setup & Index Creation (2-3 hours)
**ADR:** ADR-012  
**Date:** 2025-10-01

## Inputs
- Pinecone account credentials
- Index configuration requirements

## Outputs
- Pinecone serverless index created
- Connection credentials stored in Key Vault
- Basic health checks passing

## Acceptance Criteria
- [ ] Pinecone account provisioned
- [ ] Index "teraffi-prod" created (1536-dim, cosine metric)
- [ ] Namespaces defined (member, brand, content, trend)
- [ ] API key stored securely in Azure Key Vault
- [ ] Connection test passes from application
- [ ] Rate limits and quotas documented

## Implementation Details
Create Pinecone serverless index in us-east-1 (match app region). Configure for 1536 dimensions (OpenAI text-embedding-3-large reduced). Set up namespaces for entity types. Store credentials securely.

## Files to Create
- infrastructure/pinecone/setup.py (or .ts)
- packages/embedding-service/src/pinecone-client.ts
- docs/setup/pinecone_configuration.md

## Dependencies
ADR-014 (LLM Gateway for embeddings), Azure Key Vault