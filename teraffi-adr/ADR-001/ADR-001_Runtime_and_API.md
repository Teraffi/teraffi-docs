# ADR-001: Runtime & API Architecture (Node.js 20 + Fastify + OpenAPI 3.1)

**Status:** Accepted  
**Date:** 2025-10-02  
**Decision Makers:** Platform Architecture Team  
**Related ADRs:** ADR-002 (Workflows), ADR-003 (Messaging), ADR-004 (Auth), ADR-006 (Tenant)

---

## Context
We need a high-throughput public/admin API surface and an internal RPC layer that provide:
- **Typed contracts** with drift protection
- **Low latency** for read-heavy storefront traffic
- **Deterministic validation** and consistent error envelopes
- **First-class observability** (traces, metrics, logs) with tenant correlation
- **Scale-out** across cells with zero-downtime deploys

### Requirements
**Functional**
1. RESTful public/admin APIs with OpenAPI 3.1 source-of-truth.
2. Internal RPC for service-to-service calls (strongly typed, private).
3. Health (`/healthz`), readiness (`/readyz`), and version endpoints.
4. Pagination, filtering, and idempotent POST semantics.
5. Content negotiation (JSON only), consistent error format.

**Non-Functional**
1. **Performance:** cached GET p95 < 30ms; typical POST p95 < 120ms at 10k RPS/cell.
2. **Resilience:** circuit-breaking and timeouts on all outbound calls.
3. **Observability:** 100% trace context propagation; sampled tail-based.
4. **Governance:** schema drift fails CI; breaking changes require versioning.
5. **Security:** strict input validation; rate limits at edge and app layer.

---

## Decision
Adopt **Node.js 20 + TypeScript** with **Fastify** as the HTTP runtime, **AJV** for validation, and **OpenAPI 3.1** generated from a **single source-of-truth router registry**. Expose **tRPC** internally on `/_internal` for type-safe intra-cell RPC. Instrument with **OpenTelemetry** → Azure Application Insights.

---

## Architecture
```mermaid
flowchart LR
  A[Azure Front Door/WAF] --> B[Cell Ingress]
  B --> C[Fastify API (Node 20, TS)]
  C -->|OpenAPI Router| C1[Handlers]
  C -->|tRPC /_internal| C2[Service RPC]
  C -->|OTel| C3[App Insights]
  C -->|Auth| C4[JWT/HMAC Middlewares]
  C -->|DB| D[(PostgreSQL)]
  C -->|Cache| E[(Redis)]
  C -->|Bus| F[[Azure Service Bus]]
  C -->|Workflows| G[[Temporal]]
```
... (content truncated for brevity in code cell — full content continues below in files)

## Rationale
- **Fastify + AJV**: top-tier perf, native JSON schema validation.
- **OpenAPI 3.1**: contract-first; consumer docs; CI drift detection.
- **tRPC**: internal-only, removes boilerplate DTOs for service RPC.
- **OpenTelemetry**: vendor-neutral, end-to-end correlation.

## Implementation Details

### 1) Contract & Validation
- **Source-of-truth router registry** (TypeScript) generates:
  - OpenAPI 3.1 document (`/openapi.json`)
  - AJV validators per route
  - Type-safe handler signatures

**Error Envelope**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "price must be >= 0",
    "details": [{"path":"/price","rule":"minimum"}],
    "requestId": "req_123",
    "tenantId": "ten_123"
  }
}
```

**Pagination Envelope**
```json
{
  "data": [],
  "page": {"cursor": "abc", "has_next": true}
}
```

### 2) Middleware
- Request ID, tenant extraction, OTel context, JSON limit guard, rate limit guard (per tenant/route), idempotency guard for unsafe verbs.

### 3) OpenAPI Governance (CI)
- `make openapi:gen` → commit `openapi.json`.
- PR gate compares generated vs. tracked; drift fails.
- `semver` bump required on breaking changes; changelog bot posts endpoint diffs.

### 4) Versioning
- **URL versioning** (`/v1/…`), with deprecation headers and sunset dates.
- Backward-compatible changes allowed without version bump.

### 5) Internal RPC (tRPC)
- Namespaced under `/_internal`, private network only.
- Auth via mTLS + short-lived service tokens.
- Strict timeouts + retries; no recursive fan-out allowed.

### 6) Observability
- OTel auto-instrumentation for HTTP, Redis, Postgres, Service Bus.
- **Attributes:** `tenant_id`, `route`, `status`, `db.op`, `cache.hit`.
- Tail-based sampling target: 5% overall, 100% for errors.

## Metrics & SLOs
- **API latency:** p95 GET < 30ms (cached), POST < 120ms.
- **Error rate:** < 0.3% 5m window.
- **Contract drift:** 0 occurrences/month.
- **Trace propagation failures:** < 0.1% requests.

## Operational Procedures
- **Blue/green deploys** per cell.
- **Feature flags** for risky endpoints.
- **Runbook:** schema drift failure; 429 storms; AJV perf spikes.

## Security
- Strict AJV schemas; reject unknown fields.
- JWT for admin; HMAC publishable key for storefront (see ADR-004).
- Deny `application/*+json` smuggling; enforce `application/json`.
- 5MB JSON max; 413 on exceed.

## Alternatives
- Koa/Express: slower; more boilerplate.
- gRPC public APIs: browsers unfriendly; REST best for storefronts.

## Consequences
- **Positive:** performance, consistency, strong contracts.
- **Negative:** discipline to maintain OpenAPI; mitigated by CI.

## Success Metrics
- Gate-1 k6 passes; 0 contract drifts; P95 met in staging & prod.

## Migration/Phasing
- Phase 1: Core routes + OpenAPI gen + CI gates.
- Phase 2: Expand to all services; tRPC rollout; OTel dashboards.
- Phase 3: Tail-based sampling + advanced SLO alerts.
