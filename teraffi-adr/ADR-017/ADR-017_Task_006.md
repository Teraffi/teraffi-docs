# Task: DocuSign Integration (6-8 hours)
**ADR:** ADR-017  
**Date:** 2025-10-01

## Inputs
- DocuSign API credentials
- Contract documents
- Signer information

## Outputs
- DocuSign service
- Envelope creation
- Webhook handling
- Signed document storage

## Acceptance Criteria
- [ ] Create DocuSign envelopes with documents
- [ ] Configure signers and routing
- [ ] Set up signature tabs/anchors
- [ ] Register webhook for completion events
- [ ] Handle envelope-completed webhook
- [ ] Download signed documents
- [ ] Trigger state transition to 'signed'
- [ ] Store envelope IDs
- [ ] Unit tests with mocked DocuSign

## Implementation Details
Wrap DocuSign SDK for envelope creation. Configure signers with proper routing. Set up webhook endpoint to receive completion notifications. Download and store signed documents.

## Files to Create
- packages/deal-flow/src/docusign-service.ts
- packages/deal-flow/src/webhooks/docusign-handler.ts
- packages/api/src/routes/webhooks/docusign.ts
- packages/deal-flow/test/docusign.spec.ts

## Dependencies
Task 017-002, Task 017-004, DocuSign API account