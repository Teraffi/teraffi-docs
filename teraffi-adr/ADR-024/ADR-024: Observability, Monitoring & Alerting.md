# ADR-024: Observability, Monitoring & Alerting

**Status:** Proposed  
**Date:** 2025-10-01  
**Decision Makers:** Platform Architecture Team  
**Related ADRs:** ADR-001 (Runtime), ADR-007 (PostgreSQL), ADR-011 (Neo4j)

---

## Context

TERAFFI is a mission-critical platform managing partnerships worth millions of dollars. The operations team must have complete visibility into system health, performance, and business metrics to ensure reliability and quickly resolve issues.

**Observability Needs:**

**Infrastructure Monitoring:**
- Server health (CPU, memory, disk, network)
- Database performance (query latency, connection pools)
- Cache hit rates (Redis)
- Queue depth and processing rates
- Network latency and errors

**Application Monitoring:**
- Request rates and latency (p50, p95, p99)
- Error rates and types
- API endpoint performance
- Background job execution
- Feature usage tracking

**Business Metrics:**
- Partnerships created per day
- Affinity calculations completed
- Search queries per second
- Active users online
- Revenue metrics (MRR, churn)

**Distributed Tracing:**
- End-to-end request tracing
- Service dependency mapping
- Bottleneck identification
- Error propagation tracking

**Logging:**
- Centralized log aggregation
- Structured JSON logging
- Log search and filtering
- Error stack traces
- Audit trail

### Current Gap

Without observability:
- **Blind Operations**: No visibility into system health
- **Slow Incident Response**: Can't quickly identify root causes
- **No Proactive Alerts**: Issues discovered by users, not monitoring
- **Performance Mysteries**: Can't identify bottlenecks
- **Compliance Risk**: No audit trail for SOC 2

### Requirements

**Functional:**
1. Metrics collection and storage
2. Real-time dashboards
3. Alerting with escalation
4. Distributed tracing
5. Centralized logging
6. Log search and analysis
7. Custom business metrics
8. Anomaly detection

**Non-Functional:**
1. <1% performance overhead
2. 90 days metric retention (1 year for compliance)
3. <5 minute alerting delay
4. 99.9% monitoring system uptime
5. Support 100k+ metrics per second
6. <500ms dashboard load times

---

## Decision

**Implement comprehensive observability stack** with:
1. **Prometheus** for metrics collection and storage
2. **Grafana** for visualization and dashboards
3. **Loki** for log aggregation
4. **Tempo** for distributed tracing
5. **OpenTelemetry** for instrumentation
6. **Alertmanager** for alert routing and escalation
7. **PagerDuty** for on-call management

**Architecture:**
```
Application Services
    ↓
[OpenTelemetry SDK]
    ↓
┌─────────────────────────┐
│ Metrics → Prometheus    │
│ Logs → Loki             │
│ Traces → Tempo          │
└─────────────────────────┘
    ↓
Grafana (Unified Visualization)
    ↓
Alertmanager → PagerDuty → On-Call Engineer
```

---

## Rationale

### Why Prometheus

**Benefits:**
- Industry standard for metrics
- Pull-based model (services expose /metrics endpoint)
- Powerful query language (PromQL)
- Service discovery
- Efficient time-series storage

**Alternatives:**
- **Datadog**: Expensive SaaS, vendor lock-in
- **CloudWatch**: AWS-only, limited query capabilities
- **Decision**: Prometheus for cost and flexibility

### Why Grafana

**Benefits:**
- Beautiful, customizable dashboards
- Supports multiple data sources
- Alerting built-in
- Template variables for dynamic dashboards
- Open source, large community

### Why OpenTelemetry

**Benefits:**
- Vendor-neutral instrumentation
- Single SDK for metrics, logs, traces
- Auto-instrumentation for popular frameworks
- Future-proof (industry standard)

### Grafana Stack vs ELK Stack

**Grafana Stack (Prometheus, Loki, Tempo):**
- **Pros**: Unified UI, simpler, cheaper, better for metrics
- **Cons**: Loki less mature than Elasticsearch

**ELK Stack (Elasticsearch, Logstash, Kibana):**
- **Pros**: Mature, powerful log search
- **Cons**: Complex, expensive, resource-heavy

**Decision**: Grafana stack for simplicity and cost

---

## Implementation Details

### 1. OpenTelemetry Instrumentation

```typescript
// packages/core/src/telemetry/otel-setup.ts

import { NodeSDK } from '@opentelemetry/sdk-node';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';
import { PrometheusExporter } from '@opentelemetry/exporter-prometheus';
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http';
import { OTLPLogExporter } from '@opentelemetry/exporter-logs-otlp-http';
import { Resource } from '@opentelemetry/resources';
import { SemanticResourceAttributes } from '@opentelemetry/semantic-conventions';

export function initializeTelemetry(serviceName: string): NodeSDK {
  const sdk = new NodeSDK({
    resource: new Resource({
      [SemanticResourceAttributes.SERVICE_NAME]: serviceName,
      [SemanticResourceAttributes.SERVICE_VERSION]: process.env.APP_VERSION || 'unknown',
      [SemanticResourceAttributes.DEPLOYMENT_ENVIRONMENT]: process.env.NODE_ENV || 'development'
    }),

    // Metrics
    metricReader: new PrometheusExporter({
      port: 9464,  // Expose metrics on :9464/metrics
      endpoint: '/metrics'
    }),

    // Traces
    traceExporter: new OTLPTraceExporter({
      url: process.env.OTEL_EXPORTER_OTLP_TRACES_ENDPOINT || 'http://tempo:4318/v1/traces'
    }),

    // Logs
    logRecordProcessor: new BatchLogRecordProcessor(
      new OTLPLogExporter({
        url: process.env.OTEL_EXPORTER_OTLP_LOGS_ENDPOINT || 'http://loki:3100/loki/api/v1/push'
      })
    ),

    // Auto-instrumentation
    instrumentations: [
      getNodeAutoInstrumentations({
        '@opentelemetry/instrumentation-fs': {
          enabled: false  // Too noisy
        },
        '@opentelemetry/instrumentation-http': {
          enabled: true,
          ignoreIncomingPaths: ['/health', '/metrics']
        },
        '@opentelemetry/instrumentation-express': {
          enabled: true
        },
        '@opentelemetry/instrumentation-pg': {
          enabled: true,
          enhancedDatabaseReporting: true
        },
        '@opentelemetry/instrumentation-redis': {
          enabled: true
        }
      })
    ]
  });

  sdk.start();

  // Graceful shutdown
  process.on('SIGTERM', () => {
    sdk.shutdown()
      .then(() => console.log('Telemetry shut down'))
      .catch((error) => console.error('Error shutting down telemetry', error))
      .finally(() => process.exit(0));
  });

  return sdk;
}
```

### 2. Custom Metrics

```typescript
// packages/core/src/telemetry/metrics.ts

import { metrics } from '@opentelemetry/api';

const meter = metrics.getMeter('teraffi-platform');

// Counters
export const partnershipCreatedCounter = meter.createCounter('partnerships.created', {
  description: 'Total partnerships created',
  unit: '1'
});

export const affinityCalculationCounter = meter.createCounter('affinity.calculations', {
  description: 'Affinity calculations performed',
  unit: '1'
});

export const searchQueryCounter = meter.createCounter('search.queries', {
  description: 'Search queries executed',
  unit: '1'
});

// Histograms
export const searchLatencyHistogram = meter.createHistogram('search.latency', {
  description: 'Search query latency',
  unit: 'ms'
});

export const affinityCalculationDuration = meter.createHistogram('affinity.calculation.duration', {
  description: 'Time to calculate affinity score',
  unit: 'ms'
});

export const apiRequestDuration = meter.createHistogram('api.request.duration', {
  description: 'API request duration',
  unit: 'ms'
});

// Gauges
export const activeUsersGauge = meter.createObservableGauge('users.active', {
  description: 'Currently active users',
  unit: '1'
});

export const queueDepthGauge = meter.createObservableGauge('queue.depth', {
  description: 'Current queue depth',
  unit: '1'
});

// Usage example
export function recordPartnershipCreated(tenantId: string, type: string): void {
  partnershipCreatedCounter.add(1, {
    tenant_id: tenantId,
    partnership_type: type
  });
}

export function recordSearchQuery(userId: string, latencyMs: number): void {
  searchQueryCounter.add(1, {
    user_id: userId
  });
  
  searchLatencyHistogram.record(latencyMs, {
    user_id: userId
  });
}
```

### 3. Structured Logging

```typescript
// packages/core/src/logging/logger.ts

import pino from 'pino';
import { trace, context } from '@opentelemetry/api';

export const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  formatters: {
    level: (label) => {
      return { level: label.toUpperCase() };
    },
    bindings: (bindings) => {
      return {
        pid: bindings.pid,
        hostname: bindings.hostname,
        service: process.env.SERVICE_NAME || 'teraffi-api'
      };
    }
  },
  mixin() {
    // Add trace context to every log
    const span = trace.getSpan(context.active());
    if (!span) return {};
    
    const spanContext = span.spanContext();
    return {
      trace_id: spanContext.traceId,
      span_id: spanContext.spanId,
      trace_flags: spanContext.traceFlags
    };
  },
  serializers: {
    req: pino.stdSerializers.req,
    res: pino.stdSerializers.res,
    err: pino.stdSerializers.err
  }
});

// Convenience methods
export function logError(error: Error, context?: Record<string, any>): void {
  logger.error({
    err: error,
    ...context
  }, error.message);
}

export function logAudit(action: string, userId: string, details: Record<string, any>): void {
  logger.info({
    audit: true,
    action,
    user_id: userId,
    ...details
  }, `Audit: ${action}`);
}

export function logPerformance(operation: string, durationMs: number, metadata?: Record<string, any>): void {
  logger.info({
    performance: true,
    operation,
    duration_ms: durationMs,
    ...metadata
  }, `Performance: ${operation} took ${durationMs}ms`);
}
```

### 4. Prometheus Metrics Endpoint

```typescript
// packages/api/src/middleware/metrics.ts

import { FastifyPluginAsync } from 'fastify';
import client from 'prom-client';

// Initialize Prometheus registry
const register = new client.Registry();

// Default metrics (CPU, memory, etc.)
client.collectDefaultMetrics({ register });

// Custom metrics
export const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.001, 0.005, 0.01, 0.05, 0.1, 0.5, 1, 5]
});

export const httpRequestTotal = new client.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

export const activeConnections = new client.Gauge({
  name: 'http_active_connections',
  help: 'Number of active HTTP connections'
});

register.registerMetric(httpRequestDuration);
register.registerMetric(httpRequestTotal);
register.registerMetric(activeConnections);

// Metrics middleware
export const metricsMiddleware: FastifyPluginAsync = async (fastify) => {
  fastify.addHook('onRequest', async (request, reply) => {
    request.startTime = Date.now();
    activeConnections.inc();
  });

  fastify.addHook('onResponse', async (request, reply) => {
    const duration = (Date.now() - request.startTime!) / 1000;
    
    const labels = {
      method: request.method,
      route: request.routerPath || 'unknown',
      status_code: reply.statusCode.toString()
    };

    httpRequestDuration.observe(labels, duration);
    httpRequestTotal.inc(labels);
    activeConnections.dec();
  });

  // Expose metrics endpoint
  fastify.get('/metrics', async (request, reply) => {
    reply.type('text/plain');
    return register.metrics();
  });
};
```

### 5. Prometheus Configuration

```yaml
# infrastructure/prometheus/prometheus.yml

global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: 'teraffi-production'
    environment: 'production'

# Scrape configs
scrape_configs:
  # API services
  - job_name: 'teraffi-api'
    static_configs:
      - targets: 
        - 'api-1:9464'
        - 'api-2:9464'
        - 'api-3:9464'
    relabel_configs:
      - source_labels: [__address__]
        target_label: instance

  # PostgreSQL exporter
  - job_name: 'postgresql'
    static_configs:
      - targets: ['postgres-exporter:9187']

  # Redis exporter
  - job_name: 'redis'
    static_configs:
      - targets: ['redis-exporter:9121']

  # Neo4j metrics
  - job_name: 'neo4j'
    static_configs:
      - targets: ['neo4j:2004']
    metrics_path: '/metrics'

  # Node exporter (system metrics)
  - job_name: 'node'
    static_configs:
      - targets: 
        - 'node-exporter-1:9100'
        - 'node-exporter-2:9100'

  # Elasticsearch exporter
  - job_name: 'elasticsearch'
    static_configs:
      - targets: ['elasticsearch-exporter:9114']

# Alerting
alerting:
  alertmanagers:
    - static_configs:
      - targets: ['alertmanager:9093']

# Alert rules
rule_files:
  - '/etc/prometheus/alerts/*.yml'
```

### 6. Alert Rules

```yaml
# infrastructure/prometheus/alerts/api-alerts.yml

groups:
  - name: api_alerts
    interval: 30s
    rules:
      # High error rate
      - alert: HighErrorRate
        expr: |
          rate(http_requests_total{status_code=~"5.."}[5m])
          / rate(http_requests_total[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
          component: api
        annotations:
          summary: "High API error rate detected"
          description: "{{ $labels.instance }} has {{ $value | humanizePercentage }} error rate"

      # High latency
      - alert: HighLatency
        expr: |
          histogram_quantile(0.95, 
            rate(http_request_duration_seconds_bucket[5m])
          ) > 1.0
        for: 10m
        labels:
          severity: warning
          component: api
        annotations:
          summary: "High API latency detected"
          description: "{{ $labels.instance }} p95 latency is {{ $value }}s"

      # Service down
      - alert: ServiceDown
        expr: up == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Service is down"
          description: "{{ $labels.job }} on {{ $labels.instance }} is down"

  - name: database_alerts
    interval: 30s
    rules:
      # High database connections
      - alert: HighDatabaseConnections
        expr: |
          pg_stat_database_numbackends
          / pg_settings_max_connections > 0.8
        for: 5m
        labels:
          severity: warning
          component: database
        annotations:
          summary: "Database connection pool near capacity"
          description: "{{ $labels.instance }} using {{ $value | humanizePercentage }} of max connections"

      # Slow queries
      - alert: SlowQueries
        expr: |
          rate(pg_stat_statements_mean_exec_time[5m]) > 1000
        for: 10m
        labels:
          severity: warning
          component: database
        annotations:
          summary: "Slow database queries detected"
          description: "Average query time is {{ $value }}ms"

      # Database disk space
      - alert: DatabaseDiskSpaceLow
        expr: |
          (node_filesystem_avail_bytes{mountpoint="/data"} 
          / node_filesystem_size_bytes{mountpoint="/data"}) < 0.1
        for: 5m
        labels:
          severity: critical
          component: database
        annotations:
          summary: "Database disk space low"
          description: "Only {{ $value | humanizePercentage }} disk space remaining"

  - name: business_alerts
    interval: 1m
    rules:
      # Partnership creation rate drop
      - alert: PartnershipCreationRateDrop
        expr: |
          rate(partnerships_created_total[1h]) < 
          0.5 * avg_over_time(rate(partnerships_created_total[1h])[7d:1h])
        for: 2h
        labels:
          severity: warning
          component: business
        annotations:
          summary: "Partnership creation rate dropped significantly"
          description: "Current rate is 50% below 7-day average"

      # Affinity calculation failures
      - alert: HighAffinityCalculationFailures
        expr: |
          rate(affinity_calculations_failed_total[5m])
          / rate(affinity_calculations_total[5m]) > 0.1
        for: 10m
        labels:
          severity: warning
          component: affinity_engine
        annotations:
          summary: "High affinity calculation failure rate"
          description: "{{ $value | humanizePercentage }} of calculations failing"
```

### 7. Alertmanager Configuration

```yaml
# infrastructure/alertmanager/alertmanager.yml

global:
  resolve_timeout: 5m
  pagerduty_url: 'https://events.pagerduty.com/v2/enqueue'

# Alert routing
route:
  group_by: ['alertname', 'cluster', 'component']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  receiver: 'pagerduty'
  
  routes:
    # Critical alerts go to PagerDuty immediately
    - match:
        severity: critical
      receiver: 'pagerduty'
      group_wait: 0s
      continue: true

    # Send all alerts to Slack
    - match_re:
        severity: (critical|warning)
      receiver: 'slack'
      continue: true

    # Business alerts to different Slack channel
    - match:
        component: business
      receiver: 'slack-business'

# Receivers
receivers:
  - name: 'pagerduty'
    pagerduty_configs:
      - service_key: '<PAGERDUTY_SERVICE_KEY>'
        description: '{{ .GroupLabels.alertname }} - {{ .CommonAnnotations.summary }}'
        details:
          firing: '{{ .Alerts.Firing | len }}'
          resolved: '{{ .Alerts.Resolved | len }}'

  - name: 'slack'
    slack_configs:
      - api_url: '<SLACK_WEBHOOK_URL>'
        channel: '#alerts-production'
        title: '{{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
        color: '{{ if eq .Status "firing" }}danger{{ else }}good{{ end }}'

  - name: 'slack-business'
    slack_configs:
      - api_url: '<SLACK_WEBHOOK_URL>'
        channel: '#alerts-business'
        title: 'Business Metric Alert'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'

# Inhibition rules (suppress lower severity if higher exists)
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'cluster', 'instance']
```

### 8. Grafana Dashboard

```json
// infrastructure/grafana/dashboards/api-overview.json

{
  "dashboard": {
    "title": "API Overview",
    "tags": ["api", "production"],
    "timezone": "browser",
    "panels": [
      {
        "id": 1,
        "title": "Request Rate",
        "type": "graph",
        "targets": [{
          "expr": "sum(rate(http_requests_total[5m])) by (method, route)",
          "legendFormat": "{{method}} {{route}}"
        }],
        "yaxes": [{
          "format": "reqps",
          "label": "Requests/sec"
        }]
      },
      {
        "id": 2,
        "title": "Error Rate",
        "type": "graph",
        "targets": [{
          "expr": "sum(rate(http_requests_total{status_code=~\"5..\"}[5m])) / sum(rate(http_requests_total[5m]))",
          "legendFormat": "Error Rate"
        }],
        "yaxes": [{
          "format": "percentunit",
          "max": 0.1
        }],
        "alert": {
          "conditions": [{
            "evaluator": {
              "params": [0.05],
              "type": "gt"
            },
            "operator": {
              "type": "and"
            },
            "query": {
              "params": ["A", "5m", "now"]
            },
            "reducer": {
              "type": "avg"
            },
            "type": "query"
          }],
          "frequency": "1m",
          "handler": 1,
          "name": "High Error Rate"
        }
      },
      {
        "id": 3,
        "title": "Latency (p95)",
        "type": "graph",
        "targets": [{
          "expr": "histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, route))",
          "legendFormat": "{{route}}"
        }],
        "yaxes": [{
          "format": "s",
          "label": "Latency"
        }]
      },
      {
        "id": 4,
        "title": "Active Connections",
        "type": "stat",
        "targets": [{
          "expr": "sum(http_active_connections)"
        }]
      }
    ]
  }
}
```

---

## Migration Path

### Phase 1: Foundation (Months 1-2)
- Prometheus deployment
- Basic metrics collection
- Grafana setup
- Initial dashboards

### Phase 2: Comprehensive (Months 3-4)
- OpenTelemetry instrumentation
- Distributed tracing
- Log aggregation (Loki)
- Alert rules

### Phase 3: Advanced (Months 5-6)
- Business metric dashboards
- Anomaly detection
- Custom exporters
- Performance profiling

### Phase 4: Intelligence (Months 7-8)
- Predictive alerting
- Capacity planning
- Cost optimization tracking
- SLO tracking

---

## Success Metrics

**Reliability:**
- MTTD (Mean Time To Detect) <5 minutes
- MTTR (Mean Time To Resolve) <30 minutes
- False positive rate <5%
- Alert coverage >95% of incidents

**Performance:**
- Dashboard load time <500ms
- Query response time <2 seconds
- Metric ingestion lag <30 seconds
- 99.9% monitoring uptime

**Adoption:**
- 100% services instrumented
- >90% alerts actionable
- <2% ignored alerts
- Weekly dashboard views by all engineers

---

## Task Dependencies

001 (Prometheus) → 003 (OpenTelemetry) → 004 (Custom Metrics) → 005 (Metrics Endpoint)
                → 002 (Grafana)
                → 009 (DB Exporters)
                → 010 (Node Exporter)
                → 015 (Alert Rules) → 016 (Alertmanager) → 017 (PagerDuty)
                                                         → 018 (Slack)

003 → 006 (Logging) → 007 (Loki)
003 → 008 (Tempo)

002 → 011 (API Dashboard)
     → 012 (DB Dashboard)
     → 013 (Business Dashboard)
     → 014 (System Dashboard)
     → 021 (SLO Tracking)

009 → 020 (Query Profiling)
015 → 022 (Runbooks)

All → 019 (Instrumentation)
All → 023 (Test Suite)
All → 024 (Documentation)

## References

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [OpenTelemetry Documentation](https://opentelemetry.io/docs/)
- [Google SRE Book - Monitoring](https://sre.google/sre-book/monitoring-distributed-systems/)

---
