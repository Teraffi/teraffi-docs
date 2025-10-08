# Task: Security & Audit Logging (2-3 hours)
**ADR:** ADR-014  
**Date:** 2025-10-01

## Inputs
- Request/response data
- User/tenant context
- Security requirements

## Outputs
- Audit logger
- PII filtering (optional)
- API key rotation procedures

## Acceptance Criteria
- [ ] Log all LLM requests with user/tenant context
- [ ] Store prompt/response previews
- [ ] Track token usage and costs per request
- [ ] Optional PII filtering
- [ ] API key rotation documentation
- [ ] Compliance-ready audit trail

## Implementation Details
Log request metadata (user, tenant, model, tokens, cost) to audit table. Optionally filter PII from logged content. Document API key rotation procedure.

## Files to Create
- packages/llm-gateway/src/audit-logger.ts
- packages/llm-gateway/src/pii-filter.ts
- db/migrations/XXX_create_llm_audit_log.sql
- docs/security/api_key_rotation.md

## Dependencies
Task 014-008