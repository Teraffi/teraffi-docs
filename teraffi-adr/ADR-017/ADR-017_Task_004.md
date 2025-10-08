# Task: Document Management Service (4-5 hours)
**ADR:** ADR-017  
**Date:** 2025-10-01

## Inputs
- File uploads
- Azure Blob Storage
- Document metadata

## Outputs
- Document service
- Upload/download operations
- Version control

## Acceptance Criteria
- [ ] Upload documents to Azure Blob Storage
- [ ] Download documents with signed URLs
- [ ] Store metadata in PostgreSQL
- [ ] Support multiple document types (proposals, contracts, attachments)
- [ ] Version control for updated documents
- [ ] Access control (only deal participants)
- [ ] File type validation
- [ ] Unit tests

## Implementation Details
Implement document service wrapping Azure Blob Storage. Generate unique paths per deal. Store metadata with access control. Provide signed URLs for secure downloads.

## Files to Create
- packages/deal-flow/src/document-service.ts
- packages/deal-flow/src/storage/blob-storage.ts
- packages/deal-flow/test/document-service.spec.ts

## Dependencies
Task 017-001, Azure Blob Storage