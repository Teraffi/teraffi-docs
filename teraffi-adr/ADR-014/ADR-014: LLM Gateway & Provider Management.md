# ADR-014: LLM Gateway & Provider Management

**Status:** Proposed  
**Date:** 2025-10-01  
**Decision Makers:** Platform Architecture Team  
**Related ADRs:** ADR-012 (Embeddings), ADR-013 (GraphRAG), ADR-015 (Affinity Scoring)

---

## Context

TERAFFI's AI capabilities require Large Language Model (LLM) access for multiple use cases:

**Text Generation:**
- Query understanding (parse natural language to structured intent)
- Answer synthesis (GraphRAG responses with citations)
- Content recommendations (personalized partnership suggestions)
- Report generation (executive summaries, trend analysis)

**Embeddings:**
- Semantic search (brand/member/content similarity)
- Affinity scoring (cultural alignment detection)
- Content classification (automatic tagging)

**Analysis:**
- Cultural trend analysis (emerging movement detection)
- Partnership brief evaluation (compatibility scoring)
- Sentiment analysis (brand perception)

**Requirements:**

**Functional:**
1. Support multiple LLM providers (OpenAI, Anthropic, local Ollama)
2. Unified API abstraction (switch providers without code changes)
3. Cost tracking per provider, model, and use case
4. Automatic failover (if primary provider down, use secondary)
5. Response caching (identical prompts return cached responses)
6. Rate limiting (respect provider quotas)
7. Token counting and budget management

**Non-Functional:**
1. Latency: p95 < 3 seconds for completions, <1 second for embeddings
2. Availability: 99.9% (failover ensures continuity)
3. Cost visibility: real-time tracking, budget alerts
4. Security: API keys in Key Vault, request/response logging for audit

---

## Decision

**Build a unified LLM Gateway service** that abstracts multiple LLM providers behind a common interface, with intelligent routing, caching, cost tracking, and failover.

**Supported Providers:**
- **OpenAI** (GPT-4o, GPT-4o-mini, text-embedding-3-large) - Primary for production
- **Anthropic** (Claude 3.5 Sonnet, Claude 3 Haiku) - Secondary for failover and A/B testing
- **Ollama** (Llama 3.1, Mistral) - Local/self-hosted for development and cost-sensitive workloads

**Architecture:**
```
Application Code
    ↓
LLM Gateway (Unified Interface)
    ↓
Provider Router (selects based on model, cost, availability)
    ↓
├─ OpenAI Adapter
├─ Anthropic Adapter  
└─ Ollama Adapter
    ↓
Response Cache (Redis)
Cost Tracker (PostgreSQL)
```

---

## Rationale

### Why Gateway Pattern Over Direct Calls

**Gateway benefits:**
- **Provider independence**: Switch providers without changing application code
- **Cost optimization**: Route to cheapest provider meeting requirements
- **Resilience**: Automatic failover if provider unavailable
- **Caching**: Deduplicates identical requests (30-50% cost savings)
- **Observability**: Centralized monitoring, cost tracking, audit logs

**Direct calls drawbacks:**
- Vendor lock-in (OpenAI-specific code throughout application)
- No fallback if provider down
- Duplicate cache logic in every service
- Fragmented cost tracking
- Hard to A/B test providers

### Why Multiple Providers

**OpenAI (Primary):**
- Best quality for complex reasoning (GPT-4o)
- Fast and cheap for simple tasks (GPT-4o-mini)
- Excellent embeddings (text-embedding-3-large)
- Reliable uptime (99.9%+)

**Anthropic (Secondary):**
- Strong reasoning (Claude 3.5 Sonnet competitive with GPT-4o)
- Longer context windows (200k vs 128k)
- Different strengths (better at certain analysis tasks)
- Failover option if OpenAI down

**Ollama (Tertiary):**
- Cost optimization (self-hosted = no per-token charges)
- Privacy (sensitive data never leaves infrastructure)
- Development (no API costs in dev/test environments)
- Offline capability (works without internet)

### Cost Comparison

```typescript
// Cost per 1M tokens (as of Oct 2024):
OpenAI GPT-4o:          Input $2.50,  Output $10.00
OpenAI GPT-4o-mini:     Input $0.15,  Output $0.60
Anthropic Claude 3.5:   Input $3.00,  Output $15.00
Anthropic Claude 3 H:   Input $0.25,  Output $1.25
Ollama (self-hosted):   Infrastructure only (~$0.01/1M tokens amortized)

// Example query cost:
// GraphRAG query: 2000 input tokens, 500 output tokens
GPT-4o:        (2000 * $2.50 + 500 * $10.00) / 1M = $0.010
GPT-4o-mini:   (2000 * $0.15 + 500 * $0.60) / 1M = $0.0006
Claude 3.5:    (2000 * $3.00 + 500 * $15.00) / 1M = $0.0135
Ollama:        ~$0.0001 (infrastructure amortized)

// With 50% cache hit rate, effective costs halved
```

---

## Implementation Details

### Unified Interface

```typescript
// packages/llm-gateway/src/types.ts

export interface CompletionRequest {
  model: string;  // 'gpt-4o' | 'claude-3-5-sonnet' | 'llama-3.1-8b'
  messages: Message[];
  temperature?: number;
  max_tokens?: number;
  response_format?: { type: 'json_object' | 'text' };
  stream?: boolean;
  user?: string;  // For tracking/rate limiting
}

export interface Message {
  role: 'system' | 'user' | 'assistant';
  content: string;
}

export interface CompletionResponse {
  id: string;
  model: string;
  choices: Array<{
    index: number;
    message: Message;
    finish_reason: string;
  }>;
  usage: {
    prompt_tokens: number;
    completion_tokens: number;
    total_tokens: number;
  };
  cached: boolean;  // Whether response came from cache
  provider: string;  // 'openai' | 'anthropic' | 'ollama'
}

export interface EmbeddingRequest {
  model: string;  // 'text-embedding-3-large' | 'text-embedding-3-small'
  input: string | string[];
  dimensions?: number;
  user?: string;
}

export interface EmbeddingResponse {
  object: 'list';
  data: Array<{
    object: 'embedding';
    embedding: number[];
    index: number;
  }>;
  model: string;
  usage: {
    prompt_tokens: number;
    total_tokens: number;
  };
  cached: boolean;
  provider: string;
}
```

### LLM Gateway Service

```typescript
// packages/llm-gateway/src/gateway.ts

export class LLMGateway {
  constructor(
    private providers: Map<string, LLMProvider>,
    private router: ProviderRouter,
    private cache: ResponseCache,
    private costTracker: CostTracker
  ) {}

  async complete(request: CompletionRequest): Promise<CompletionResponse> {
    // 1. Check cache
    const cacheKey = this.cache.generateKey(request);
    const cached = await this.cache.get(cacheKey);
    if (cached) {
      return { ...cached, cached: true };
    }

    // 2. Select provider
    const provider = await this.router.selectProvider(request);

    // 3. Execute with retry and fallback
    let response: CompletionResponse;
    try {
      response = await this.executeWithRetry(provider, request);
    } catch (error) {
      // Fallback to secondary provider
      const fallbackProvider = await this.router.selectFallbackProvider(request);
      response = await this.executeWithRetry(fallbackProvider, request);
    }

    // 4. Cache response
    await this.cache.set(cacheKey, response, 3600);  // 1 hour TTL

    // 5. Track cost
    await this.costTracker.record({
      provider: response.provider,
      model: response.model,
      prompt_tokens: response.usage.prompt_tokens,
      completion_tokens: response.usage.completion_tokens,
      cost: this.calculateCost(response),
      timestamp: new Date()
    });

    return { ...response, cached: false };
  }

  async embed(request: EmbeddingRequest): Promise<EmbeddingResponse> {
    // Similar pattern: cache → route → execute → track
    const cacheKey = this.cache.generateKey(request);
    const cached = await this.cache.get(cacheKey);
    if (cached) return { ...cached, cached: true };

    const provider = await this.router.selectProvider(request);
    const response = await provider.embed(request);

    await this.cache.set(cacheKey, response, 86400);  // 24 hour TTL for embeddings
    await this.costTracker.record({
      provider: response.provider,
      model: response.model,
      prompt_tokens: response.usage.prompt_tokens,
      cost: this.calculateEmbeddingCost(response),
      timestamp: new Date()
    });

    return { ...response, cached: false };
  }

  private async executeWithRetry(
    provider: LLMProvider,
    request: CompletionRequest,
    maxRetries: number = 3
  ): Promise<CompletionResponse> {
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        return await provider.complete(request);
      } catch (error) {
        if (attempt === maxRetries || !this.isRetryable(error)) {
          throw error;
        }
        // Exponential backoff
        await sleep(Math.pow(2, attempt) * 1000);
      }
    }
    throw new Error('Max retries exceeded');
  }

  private isRetryable(error: any): boolean {
    // Retry on rate limits, timeouts, server errors
    return error.status === 429 || error.status >= 500 || error.code === 'ETIMEDOUT';
  }
}
```

### Provider Adapters

```typescript
// packages/llm-gateway/src/providers/openai-adapter.ts

export class OpenAIAdapter implements LLMProvider {
  constructor(private client: OpenAI) {}

  async complete(request: CompletionRequest): Promise<CompletionResponse> {
    const response = await this.client.chat.completions.create({
      model: request.model,
      messages: request.messages,
      temperature: request.temperature ?? 0.7,
      max_tokens: request.max_tokens,
      response_format: request.response_format,
      user: request.user
    });

    return {
      id: response.id,
      model: response.model,
      choices: response.choices,
      usage: response.usage,
      cached: false,
      provider: 'openai'
    };
  }

  async embed(request: EmbeddingRequest): Promise<EmbeddingResponse> {
    const response = await this.client.embeddings.create({
      model: request.model,
      input: request.input,
      dimensions: request.dimensions,
      user: request.user
    });

    return {
      object: 'list',
      data: response.data,
      model: response.model,
      usage: response.usage,
      cached: false,
      provider: 'openai'
    };
  }
}
```

```typescript
// packages/llm-gateway/src/providers/anthropic-adapter.ts

export class AnthropicAdapter implements LLMProvider {
  constructor(private client: Anthropic) {}

  async complete(request: CompletionRequest): Promise<CompletionResponse> {
    // Map model names
    const modelMap: Record<string, string> = {
      'claude-3-5-sonnet': 'claude-3-5-sonnet-20241022',
      'claude-3-haiku': 'claude-3-haiku-20240307'
    };

    // Anthropic uses different message format (system separate)
    const systemMessage = request.messages.find(m => m.role === 'system');
    const messages = request.messages.filter(m => m.role !== 'system');

    const response = await this.client.messages.create({
      model: modelMap[request.model] || request.model,
      system: systemMessage?.content,
      messages: messages.map(m => ({
        role: m.role as 'user' | 'assistant',
        content: m.content
      })),
      temperature: request.temperature ?? 0.7,
      max_tokens: request.max_tokens ?? 4096
    });

    // Normalize to OpenAI format
    return {
      id: response.id,
      model: request.model,
      choices: [{
        index: 0,
        message: {
          role: 'assistant',
          content: response.content[0].type === 'text' 
            ? response.content[0].text 
            : ''
        },
        finish_reason: response.stop_reason || 'stop'
      }],
      usage: {
        prompt_tokens: response.usage.input_tokens,
        completion_tokens: response.usage.output_tokens,
        total_tokens: response.usage.input_tokens + response.usage.output_tokens
      },
      cached: false,
      provider: 'anthropic'
    };
  }

  async embed(request: EmbeddingRequest): Promise<EmbeddingResponse> {
    // Anthropic doesn't have embedding API yet
    throw new Error('Anthropic does not support embeddings - use OpenAI');
  }
}
```

```typescript
// packages/llm-gateway/src/providers/ollama-adapter.ts

export class OllamaAdapter implements LLMProvider {
  constructor(private baseUrl: string = 'http://localhost:11434') {}

  async complete(request: CompletionRequest): Promise<CompletionResponse> {
    const response = await fetch(`${this.baseUrl}/api/chat`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        model: request.model,  // 'llama3.1', 'mistral', etc.
        messages: request.messages,
        options: {
          temperature: request.temperature ?? 0.7,
          num_predict: request.max_tokens
        },
        stream: false
      })
    });

    const data = await response.json();

    return {
      id: `ollama-${Date.now()}`,
      model: request.model,
      choices: [{
        index: 0,
        message: {
          role: 'assistant',
          content: data.message.content
        },
        finish_reason: 'stop'
      }],
      usage: {
        prompt_tokens: data.prompt_eval_count || 0,
        completion_tokens: data.eval_count || 0,
        total_tokens: (data.prompt_eval_count || 0) + (data.eval_count || 0)
      },
      cached: false,
      provider: 'ollama'
    };
  }

  async embed(request: EmbeddingRequest): Promise<EmbeddingResponse> {
    const inputs = Array.isArray(request.input) ? request.input : [request.input];
    
    const embeddings = await Promise.all(
      inputs.map(async (text, index) => {
        const response = await fetch(`${this.baseUrl}/api/embeddings`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            model: request.model || 'nomic-embed-text',
            prompt: text
          })
        });
        const data = await response.json();
        return {
          object: 'embedding' as const,
          embedding: data.embedding,
          index
        };
      })
    );

    return {
      object: 'list',
      data: embeddings,
      model: request.model || 'nomic-embed-text',
      usage: {
        prompt_tokens: inputs.join(' ').length / 4,  // Rough estimate
        total_tokens: inputs.join(' ').length / 4
      },
      cached: false,
      provider: 'ollama'
    };
  }
}
```

### Provider Router

```typescript
// packages/llm-gateway/src/router.ts

interface RoutingStrategy {
  preferredProvider: string;
  fallbackProviders: string[];
  costThreshold?: number;  // Switch to cheaper if cost important
}

export class ProviderRouter {
  private strategies: Map<string, RoutingStrategy> = new Map([
    // High-quality reasoning: OpenAI GPT-4o primary, Claude fallback
    ['gpt-4o', {
      preferredProvider: 'openai',
      fallbackProviders: ['anthropic/claude-3-5-sonnet']
    }],
    // Fast/cheap tasks: GPT-4o-mini primary, Ollama fallback
    ['gpt-4o-mini', {
      preferredProvider: 'openai',
      fallbackProviders: ['ollama/llama3.1']
    }],
    // Embeddings: OpenAI only (best quality)
    ['text-embedding-3-large', {
      preferredProvider: 'openai',
      fallbackProviders: []
    }]
  ]);

  async selectProvider(request: CompletionRequest | EmbeddingRequest): Promise<LLMProvider> {
    const strategy = this.strategies.get(request.model);
    if (!strategy) {
      throw new Error(`No routing strategy for model: ${request.model}`);
    }

    // Check if preferred provider available
    const provider = this.providers.get(strategy.preferredProvider);
    if (provider && await this.isAvailable(provider)) {
      return provider;
    }

    // Fallback to next available provider
    for (const fallbackName of strategy.fallbackProviders) {
      const fallbackProvider = this.providers.get(fallbackName);
      if (fallbackProvider && await this.isAvailable(fallbackProvider)) {
        return fallbackProvider;
      }
    }

    throw new Error(`No available providers for model: ${request.model}`);
  }

  async selectFallbackProvider(request: CompletionRequest): Promise<LLMProvider> {
    const strategy = this.strategies.get(request.model);
    if (!strategy || strategy.fallbackProviders.length === 0) {
      throw new Error(`No fallback available for model: ${request.model}`);
    }

    for (const fallbackName of strategy.fallbackProviders) {
      const provider = this.providers.get(fallbackName);
      if (provider && await this.isAvailable(provider)) {
        return provider;
      }
    }

    throw new Error(`All fallback providers unavailable`);
  }

  private async isAvailable(provider: LLMProvider): Promise<boolean> {
    // Health check: try simple request or ping endpoint
    try {
      await provider.healthCheck();
      return true;
    } catch {
      return false;
    }
  }
}
```

### Response Caching

```typescript
// packages/llm-gateway/src/cache.ts

export class ResponseCache {
  constructor(private redis: RedisClient) {}

  generateKey(request: CompletionRequest | EmbeddingRequest): string {
    // Create stable hash from request (model + messages + params)
    const canonical = JSON.stringify({
      model: request.model,
      messages: 'messages' in request ? request.messages : undefined,
      input: 'input' in request ? request.input : undefined,
      temperature: 'temperature' in request ? request.temperature : undefined,
      max_tokens: 'max_tokens' in request ? request.max_tokens : undefined
    });
    
    return `llm:cache:${hashString(canonical)}`;
  }

  async get(key: string): Promise<CompletionResponse | EmbeddingResponse | null> {
    const cached = await this.redis.get(key);
    return cached ? JSON.parse(cached) : null;
  }

  async set(
    key: string,
    response: CompletionResponse | EmbeddingResponse,
    ttl: number
  ): Promise<void> {
    await this.redis.setex(key, ttl, JSON.stringify(response));
  }

  async invalidate(pattern: string): Promise<void> {
    // Admin function to clear cache by pattern
    const keys = await this.redis.keys(pattern);
    if (keys.length > 0) {
      await this.redis.del(...keys);
    }
  }
}
```

### Cost Tracking

```typescript
// packages/llm-gateway/src/cost-tracker.ts

interface CostRecord {
  provider: string;
  model: string;
  prompt_tokens: number;
  completion_tokens: number;
  cost: number;
  use_case?: string;  // 'graphrag' | 'embeddings' | 'analysis'
  user_id?: string;
  tenant_id?: string;
  timestamp: Date;
}

export class CostTracker {
  private costTables: Record<string, Record<string, { input: number; output: number }>> = {
    openai: {
      'gpt-4o': { input: 2.50 / 1_000_000, output: 10.00 / 1_000_000 },
      'gpt-4o-mini': { input: 0.15 / 1_000_000, output: 0.60 / 1_000_000 },
      'text-embedding-3-large': { input: 0.13 / 1_000_000, output: 0 }
    },
    anthropic: {
      'claude-3-5-sonnet': { input: 3.00 / 1_000_000, output: 15.00 / 1_000_000 },
      'claude-3-haiku': { input: 0.25 / 1_000_000, output: 1.25 / 1_000_000 }
    },
    ollama: {
      // Self-hosted: amortized infrastructure cost
      'llama3.1': { input: 0.01 / 1_000_000, output: 0.01 / 1_000_000 }
    }
  };

  constructor(private db: Database) {}

  async record(record: CostRecord): Promise<void> {
    await this.db.insertInto('llm_costs')
      .values(record)
      .execute();
  }

  calculateCost(provider: string, model: string, promptTokens: number, completionTokens: number): number {
    const rates = this.costTables[provider]?.[model];
    if (!rates) return 0;
    
    return (promptTokens * rates.input) + (completionTokens * rates.output);
  }

  async getDailyCost(date: Date = new Date()): Promise<number> {
    const result = await this.db
      .selectFrom('llm_costs')
      .select(db.fn.sum<number>('cost').as('total'))
      .where('timestamp', '>=', startOfDay(date))
      .where('timestamp', '<', startOfDay(addDays(date, 1)))
      .executeTakeFirst();
    
    return result?.total || 0;
  }

  async getCostByUseCase(startDate: Date, endDate: Date): Promise<Record<string, number>> {
    const results = await this.db
      .selectFrom('llm_costs')
      .select(['use_case', db.fn.sum<number>('cost').as('total')])
      .where('timestamp', '>=', startDate)
      .where('timestamp', '<', endDate)
      .groupBy('use_case')
      .execute();
    
    return results.reduce((acc, r) => ({
      ...acc,
      [r.use_case || 'unknown']: r.total
    }), {});
  }
}
```

---

## Database Schema

```sql
-- Track LLM API usage and costs
CREATE TABLE llm_costs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  provider VARCHAR(50) NOT NULL,  -- 'openai' | 'anthropic' | 'ollama'
  model VARCHAR(100) NOT NULL,    -- 'gpt-4o' | 'claude-3-5-sonnet' | etc.
  prompt_tokens INTEGER NOT NULL,
  completion_tokens INTEGER DEFAULT 0,
  cost DECIMAL(10,6) NOT NULL,
  use_case VARCHAR(100),          -- 'graphrag' | 'embeddings' | 'analysis'
  user_id UUID,
  tenant_id TEXT,
  timestamp TIMESTAMPTZ DEFAULT NOW(),
  
  -- Partitioning by month for efficient archival
  created_at TIMESTAMPTZ DEFAULT NOW()
) PARTITION BY RANGE (created_at);

CREATE INDEX idx_llm_costs_timestamp ON llm_costs(timestamp);
CREATE INDEX idx_llm_costs_provider_model ON llm_costs(provider, model);
CREATE INDEX idx_llm_costs_use_case ON llm_costs(use_case);
CREATE INDEX idx_llm_costs_tenant ON llm_costs(tenant_id);

-- Create monthly partitions
CREATE TABLE llm_costs_2025_10 PARTITION OF llm_costs
  FOR VALUES FROM ('2025-10-01') TO ('2025-11-01');
```

---

## Configuration

```typescript
// config/llm-gateway.ts

export const llmGatewayConfig = {
  providers: {
    openai: {
      apiKey: process.env.OPENAI_API_KEY,
      organization: process.env.OPENAI_ORG_ID,
      timeout: 30000,  // 30 seconds
      maxRetries: 3
    },
    anthropic: {
      apiKey: process.env.ANTHROPIC_API_KEY,
      timeout: 30000,
      maxRetries: 3
    },
    ollama: {
      baseUrl: process.env.OLLAMA_BASE_URL || 'http://localhost:11434',
      timeout: 60000  // Local inference can be slower
    }
  },
  
  cache: {
    completionTTL: 3600,      // 1 hour
    embeddingTTL: 86400,      // 24 hours
    maxCacheSize: '1GB'
  },
  
  costAlerts: {
    dailyBudget: 100,         // $100/day alert
    monthlyBudget: 2000,      // $2000/month alert
    alertEmail: 'ops@teraffi.io'
  },
  
  rateLimits: {
    openai: {
      requestsPerMinute: 500,
      tokensPerMinute: 150000
    },
    anthropic: {
      requestsPerMinute: 1000,
      tokensPerMinute: 100000
    }
  }
};
```

---

## Monitoring & Alerts

```typescript
// Metrics to track
interface LLMGatewayMetrics {
  // Performance
  request_latency_ms: Histogram;  // By provider, model
  cache_hit_rate: Gauge;
  provider_availability: Gauge;
  
  // Usage
  requests_total: Counter;  // By provider, model, use_case
  tokens_total: Counter;    // By provider, model, type (prompt/completion)
  
  // Cost
  cost_total_usd: Counter;  // By provider, model, use_case
  daily_cost_usd: Gauge;
  monthly_cost_usd: Gauge;
  
  // Errors
  errors_total: Counter;    // By provider, error_type
  fallback_used_total: Counter;
  retries_total: Counter;
}
```

```yaml
# Prometheus alert rules
groups:
  - name: llm_gateway_alerts
    rules:
      - alert: LLMGatewayHighLatency
        expr: histogram_quantile(0.95, llm_request_latency_ms) > 5000
        for: 5m
        annotations:
          summary: "LLM Gateway p95 latency >5s"
          
      - alert: LLMGatewayHighCost
        expr: llm_daily_cost_usd > 100
        annotations:
          summary: "Daily LLM cost exceeded $100"
          
      - alert: LLMProviderDown
        expr: llm_provider_availability < 1
        for: 2m
        annotations:
          summary: "LLM provider {{ $labels.provider }} unavailable"
          
      - alert: LLMGatewayCacheMiss
        expr: llm_cache_hit_rate < 0.3
        for: 10m
        annotations:
          summary: "LLM cache hit rate below 30%"
```

---

## Security

```typescript
// API key rotation (quarterly)
async function rotateAPIKeys() {
  // 1. Generate new keys in provider dashboard
  // 2. Store in Azure Key Vault
  // 3. Update LLM Gateway config
  // 4. Monitor for errors
  // 5. Deactivate old keys after 24h overlap
}

// Request/response logging for audit
async function logRequest(
  request: CompletionRequest,
  response: CompletionResponse,
  userId: string,
  tenantId: string
) {
  await auditLog.record({
    user_id: userId,
    tenant_id: tenantId,
    provider: response.provider,
    model: response.model,
    prompt_preview: request.messages[0]?.content.substring(0, 100),
    response_preview: response.choices[0]?.message.content.substring(0, 100),
    tokens_used: response.usage.total_tokens,
    cost: calculateCost(response),
    timestamp: new Date()
  });
}

// PII filtering (optional)
function filterPII(text: string): string {
  return text
    .replace(/\b[\w\.-]+@[\w\.-]+\.\w{2,4}\b/g, '[EMAIL]')
    .replace(/\b\d{3}[-.]?\d{3}[-.]?\d{4}\b/g, '[PHONE]')
    .replace(/\b\d{3}-\d{2}-\d{4}\b/g, '[SSN]');
}
```

---

## Migration Path

### Phase 1: Core Gateway (Months 1-2)
- Implement unified interface
- OpenAI adapter only
- Basic caching (Redis)
- Cost tracking (PostgreSQL)

### Phase 2: Multi-Provider (Months 3-4)
- Add Anthropic adapter
- Add Ollama adapter
- Provider routing and failover
- A/B testing framework

### Phase 3: Optimization (Months 5-6)
- Advanced caching strategies
- Cost optimization (route to cheapest)
- Rate limiting and quotas
- Performance tuning

### Phase 4: Enterprise (Months 7-9)
- Fine-tuning support (custom models)
- Advanced monitoring and alerts
- Compliance features (audit logs, PII filtering)
- Multi-region routing

---

## Alternatives Considered

### Alternative 1: LangChain LLM Abstraction
**Pros:** Existing framework, many integrations
**Cons:** Heavy dependency, more than we need, harder to optimize
**Decision:** Custom gateway gives more control

### Alternative 2: Direct Provider SDKs
**Pros:** Simple, no abstraction overhead
**Cons:** Vendor lock-in, no failover, fragmented cost tracking
**Decision:** Gateway worth the complexity

### Alternative 3: OpenRouter (Third-Party Gateway)
**Pros:** Managed service, handles routing
**Cons:** External dependency, additional cost, less control
**Decision:** Build our own for full control and cost optimization

---

## Consequences

### Positive
- Provider independence (easy to switch or add providers)
- Cost optimization (caching, routing, tracking)
- Resilience (automatic failover)
- Observability (centralized metrics and logging)
- Security (API keys in Key Vault, audit logging)

### Negative
- Additional service to maintain
- Abstraction overhead (slight latency increase)
- Learning curve (team must understand gateway patterns)

### Neutral
- Requires monitoring and tuning
- Cost tracking helps but doesn't reduce costs directly
- Caching effectiveness depends on query patterns

---

## Success Metrics

**Performance:**
- p95 latency: <3s completions, <1s embeddings
- Cache hit rate: >50%
- Provider availability: 99.9%

**Cost:**
- Average cost per completion: <$0.01
- Average cost per embedding: <$0.0001
- Monthly cost: <$2000 at 10k queries/day

**Reliability:**
- Failover success rate: >95%
- Error rate: <1%

---

## References

- [OpenAI API Documentation](https://platform.openai.com/docs/)
- [Anthropic Claude API](https://docs.anthropic.com/)
- [Ollama Documentation](https://ollama.ai/docs)
- [LLM Cost Tracking Best Practices](https://www.cloudflare.com/learning/ai/what-is-llm-gateway/)

---
