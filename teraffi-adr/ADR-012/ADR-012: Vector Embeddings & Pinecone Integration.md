# ADR-012: Vector Embeddings & Pinecone Integration

**Status:** Proposed  
**Date:** 2025-10-01  
**Decision Makers:** Platform Architecture Team  
**Related ADRs:** ADR-011 (Neo4j), ADR-013 (GraphRAG), ADR-014 (LLM Gateway)

---

## Context

TERAFFI's Pillar 1 (Intelligence) requires semantic understanding beyond keyword matching and structured relationships. Use cases include:

**Semantic Search:**
- "Find brands that align with sustainability values" (not just keyword "sustainability")
- "Content similar to documentaries about social entrepreneurship"
- "Partnerships like our most successful collaborations"

**Similarity Matching:**
- Brand value alignment (beyond stated keywords)
- Content theme similarity (understanding narrative/tone)
- Member interest matching (implicit from behavior, not just explicit tags)

**Contextual Recommendations:**
- "Given this brand's mission statement, find aligned creators"
- "Surface trends relevant to this partnership brief"
- "Recommend content opportunities based on past engagement"

**Why Traditional Search Fails:**
```
Query: "eco-friendly fashion brands"
Keyword match: Only finds brands with exact terms "eco-friendly" or "fashion"
Misses: "sustainable apparel", "circular economy clothing", "regenerative textiles"

Semantic understanding needed: These concepts are related even with different words.
```

**Requirements:**

**Functional:**
1. Generate embeddings for text (brand values, member goals, content descriptions, partnership briefs)
2. Store embeddings with metadata for filtering
3. Perform similarity search (cosine/euclidean distance)
4. Support hybrid search (semantic + metadata filters)
5. Update embeddings when source content changes
6. Scale to millions of vectors

**Non-Functional:**
1. Query latency p95 < 200ms
2. Indexing throughput > 1000 vectors/minute
3. 99.9% availability
4. Support for 1536-dimension vectors (OpenAI standard)
5. Cost-effective at scale

---

## Decision

**Use Pinecone as the primary vector database** with OpenAI's `text-embedding-3-large` model for generating embeddings.

**Architecture:**
- Pinecone serverless index (pod-based for Phase 2 if needed)
- Embedding generation via LLM Gateway (ADR-014)
- Async embedding pipeline: content change → queue → embed → upsert to Pinecone
- Hybrid search: Pinecone semantic + metadata filters
- Cache frequent queries in Redis (ADR-009)

---

## Rationale

### Why Pinecone Over Alternatives

**Pinecone vs. pgvector (PostgreSQL extension):**
- **Scale**: Pinecone optimized for billion-scale vectors; pgvector struggles beyond millions
- **Performance**: Pinecone HNSW index faster than pgvector IVFFlat at scale
- **Latency**: Pinecone p95 ~50ms; pgvector p95 ~200ms+ for large datasets
- **Maintenance**: Pinecone fully managed; pgvector requires tuning, VACUUM, reindexing
- **Limitation**: pgvector suitable for <1M vectors; TERAFFI will exceed this by Year 2

**Pinecone vs. Weaviate:**
- **Simplicity**: Pinecone simpler API (upsert, query); Weaviate more complex schema
- **Serverless**: Pinecone serverless = pay-per-use; Weaviate requires cluster management
- **Integration**: Pinecone native OpenAI integration; Weaviate more generic
- **Community**: Pinecone larger community, more examples

**Pinecone vs. Qdrant:**
- **Maturity**: Pinecone more mature, battle-tested at scale
- **Managed service**: Pinecone SaaS easier ops; Qdrant often self-hosted
- **Cost**: Pinecone competitive pricing at scale; Qdrant cheaper small-scale but ops overhead
- **Documentation**: Pinecone superior docs and support

**Pinecone vs. Elasticsearch with vector search:**
- **Purpose-built**: Pinecone designed for vectors; Elasticsearch general-purpose
- **Performance**: Pinecone HNSW optimized; Elasticsearch vector search added feature
- **Cost**: Pinecone cost-effective for vector-heavy workloads; Elasticsearch expensive for large indexes

**Decision:** Pinecone offers best balance of performance, scalability, ease of use, and managed operations.

### Why OpenAI text-embedding-3-large

**OpenAI vs. Open-source models (sentence-transformers):**
- **Quality**: OpenAI embeddings higher quality on diverse domains
- **Consistency**: Stable API, versioned models
- **Speed**: Hosted inference faster than self-hosted
- **Cost**: Competitive at scale ($0.13/1M tokens)

**text-embedding-3-large vs. text-embedding-3-small:**
- **Dimensions**: 3072 vs. 1536 (higher dimensionality = better semantic capture)
- **Performance**: 10-15% better retrieval accuracy
- **Cost**: Same pricing ($0.13/1M tokens)
- **Decision**: Use large model; can reduce dimensions to 1536 if storage becomes issue

**Alternative: Anthropic embeddings** - Not yet available as of Oct 2025; monitor for future

---

## Implementation Details

### Pinecone Index Configuration

```python
# Index setup (run once via admin script)
from pinecone import Pinecone, ServerlessSpec

pc = Pinecone(api_key=PINECONE_API_KEY)

# Create serverless index
pc.create_index(
    name="teraffi-prod",
    dimension=1536,  # OpenAI text-embedding-3-large (can reduce from 3072)
    metric="cosine",  # cosine similarity (most common for text)
    spec=ServerlessSpec(
        cloud="aws",
        region="us-east-1"  # Match application region
    )
)

# Namespace strategy for multi-tenancy
# - Global namespace: "members", "brands", "content", "trends"
# - Tenant-isolated namespaces: "tenant_{tenant_id}" (if needed)
```

### Embedding Pipeline

```typescript
// packages/embedding-service/src/pipeline.ts

interface EmbeddingJob {
  entity_type: 'member' | 'brand' | 'content' | 'partnership_brief';
  entity_id: string;
  text_content: string;
  metadata: Record<string, any>;
  tenant_id?: string;
}

export class EmbeddingPipeline {
  constructor(
    private llmGateway: LLMGatewayClient,
    private pinecone: PineconeClient,
    private queue: BullQueue
  ) {}

  // Triggered by events: brand.created, brand.values_updated, etc.
  async enqueueEmbedding(job: EmbeddingJob): Promise<void> {
    await this.queue.add('generate-embedding', job, {
      attempts: 3,
      backoff: { type: 'exponential', delay: 2000 }
    });
  }

  // Worker processes job
  async processEmbeddingJob(job: EmbeddingJob): Promise<void> {
    // 1. Generate embedding via LLM Gateway
    const embedding = await this.llmGateway.embed({
      model: 'text-embedding-3-large',
      input: job.text_content,
      dimensions: 1536  // Reduce from 3072 for storage efficiency
    });

    // 2. Upsert to Pinecone
    const index = this.pinecone.Index('teraffi-prod');
    await index.namespace(job.entity_type).upsert([{
      id: job.entity_id,
      values: embedding.data[0].embedding,
      metadata: {
        ...job.metadata,
        entity_type: job.entity_type,
        tenant_id: job.tenant_id,
        indexed_at: new Date().toISOString(),
        text_preview: job.text_content.substring(0, 200)  // For debugging
      }
    }]);

    // 3. Update PostgreSQL with embedding status
    await this.db.updateTable('embedding_status')
      .set({ 
        pinecone_id: job.entity_id,
        last_embedded_at: new Date(),
        embedding_version: 'text-embedding-3-large-1536'
      })
      .where('entity_type', '=', job.entity_type)
      .where('entity_id', '=', job.entity_id)
      .execute();
  }
}
```

### Event-Driven Embedding Updates

```typescript
// When brand values are updated in PostgreSQL
async function updateBrandValues(brandId: string, values: string[]) {
  await db.transaction(async (trx) => {
    // 1. Update PostgreSQL
    await trx.updateTable('brands')
      .set({ values, updated_at: new Date() })
      .where('id', '=', brandId)
      .execute();
    
    // 2. Publish event to trigger re-embedding
    await trx.insertInto('outbox')
      .values({
        topic: 'brand.values_updated',
        event_id: generateId(),
        payload: { brandId, values },
        created_at: new Date()
      })
      .execute();
  });
}

// Embedding worker consumes event
async function handleBrandValuesUpdated(event: BrandValuesUpdatedEvent) {
  const brand = await db.selectFrom('brands')
    .selectAll()
    .where('id', '=', event.brandId)
    .executeTakeFirstOrThrow();
  
  // Construct rich text representation for embedding
  const textContent = `
    Brand: ${brand.name}
    Industry: ${brand.industry}
    Values: ${brand.values.join(', ')}
    Mission: ${brand.mission_statement || 'Not provided'}
    Target Market: ${brand.target_markets?.join(', ') || 'General'}
  `.trim();

  await embeddingPipeline.enqueueEmbedding({
    entity_type: 'brand',
    entity_id: brand.id,
    text_content: textContent,
    metadata: {
      name: brand.name,
      industry: brand.industry,
      annual_revenue: brand.annual_revenue,
      founded_year: brand.founded_year
    }
  });
}
```

### Semantic Search Queries

**1. Brand Alignment Search**
```typescript
// Find brands aligned with a member's cultural interests
async function findAlignedBrands(memberId: string, options: SearchOptions = {}) {
  // 1. Get member's stated interests and past engagement
  const member = await db.selectFrom('members')
    .leftJoin('member_goals', 'members.id', 'member_goals.member_id')
    .select([
      'members.id',
      'members.industry',
      db.fn.agg<string[]>('array_agg', ['member_goals.title']).as('goals')
    ])
    .where('members.id', '=', memberId)
    .groupBy('members.id')
    .executeTakeFirstOrThrow();
  
  // 2. Construct query text
  const queryText = `
    Looking for brands in ${member.industry} industry
    Goals: ${member.goals.join(', ')}
    Seeking partnerships aligned with these objectives
  `.trim();
  
  // 3. Generate query embedding
  const queryEmbedding = await llmGateway.embed({
    model: 'text-embedding-3-large',
    input: queryText,
    dimensions: 1536
  });
  
  // 4. Query Pinecone with hybrid search (semantic + metadata filters)
  const index = pinecone.Index('teraffi-prod');
  const results = await index.namespace('brand').query({
    vector: queryEmbedding.data[0].embedding,
    topK: options.limit || 20,
    includeMetadata: true,
    filter: {
      // Metadata filters (exact match)
      ...(options.industry && { industry: { $eq: options.industry } }),
      ...(options.minRevenue && { annual_revenue: { $gte: options.minRevenue } })
    }
  });
  
  // 5. Enrich with Neo4j data (existing partnerships, mutual connections)
  const enriched = await enrichWithGraphData(results.matches);
  
  return enriched;
}
```

**2. Content Similarity Search**
```typescript
// Find content similar to a given piece (for partnership ideas)
async function findSimilarContent(contentId: string, limit: number = 10) {
  // 1. Get embedding for source content
  const index = pinecone.Index('teraffi-prod');
  const sourceVector = await index.namespace('content').fetch([contentId]);
  
  if (!sourceVector.records[contentId]) {
    throw new Error('Content not found in vector index');
  }
  
  // 2. Find similar content
  const results = await index.namespace('content').query({
    vector: sourceVector.records[contentId].values,
    topK: limit + 1,  // +1 because source will be in results
    includeMetadata: true,
    filter: {
      entity_id: { $ne: contentId }  // Exclude source content
    }
  });
  
  // 3. Return with similarity scores
  return results.matches.map(match => ({
    id: match.id,
    title: match.metadata.title,
    type: match.metadata.type,
    platform: match.metadata.platform,
    similarity_score: match.score,  // Cosine similarity (0-1)
    audience_size: match.metadata.audience_size
  }));
}
```

**3. Partnership Brief Matching**
```typescript
// Match partnership brief to suitable partners
async function matchPartnershipBrief(brief: PartnershipBrief) {
  // 1. Embed the partnership brief
  const briefText = `
    Partnership Type: ${brief.type}
    Objectives: ${brief.objectives.join(', ')}
    Target Audience: ${brief.target_demographics.join(', ')}
    Budget Range: ${brief.budget_min}-${brief.budget_max}
    Timeline: ${brief.timeline}
    Requirements: ${brief.requirements}
  `.trim();
  
  const briefEmbedding = await llmGateway.embed({
    model: 'text-embedding-3-large',
    input: briefText,
    dimensions: 1536
  });
  
  // 2. Search across multiple namespaces (brands, creators, content)
  const index = pinecone.Index('teraffi-prod');
  
  const [brandMatches, contentMatches] = await Promise.all([
    index.namespace('brand').query({
      vector: briefEmbedding.data[0].embedding,
      topK: 10,
      includeMetadata: true
    }),
    index.namespace('content').query({
      vector: briefEmbedding.data[0].embedding,
      topK: 10,
      includeMetadata: true
    })
  ]);
  
  // 3. Combine and rank by relevance + feasibility
  return rankPartnershipMatches(brandMatches, contentMatches, brief);
}
```

### Hybrid Search (Semantic + Metadata Filters)

```typescript
// Complex query: semantic similarity + hard constraints
async function advancedBrandSearch(params: {
  semanticQuery: string;
  industry?: string;
  minRevenue?: number;
  maxRevenue?: number;
  location?: string;
  foundedAfter?: number;
}) {
  // 1. Generate embedding for semantic query
  const queryEmbedding = await llmGateway.embed({
    model: 'text-embedding-3-large',
    input: params.semanticQuery,
    dimensions: 1536
  });
  
  // 2. Build metadata filter (AND conditions)
  const filter: any = {};
  if (params.industry) filter.industry = { $eq: params.industry };
  if (params.minRevenue) filter.annual_revenue = { $gte: params.minRevenue };
  if (params.maxRevenue) filter.annual_revenue = { ...filter.annual_revenue, $lte: params.maxRevenue };
  if (params.location) filter.location = { $eq: params.location };
  if (params.foundedAfter) filter.founded_year = { $gte: params.foundedAfter };
  
  // 3. Query with both semantic and metadata filters
  const index = pinecone.Index('teraffi-prod');
  const results = await index.namespace('brand').query({
    vector: queryEmbedding.data[0].embedding,
    topK: 50,
    includeMetadata: true,
    filter: Object.keys(filter).length > 0 ? filter : undefined
  });
  
  return results.matches;
}
```

---

## Data Model

### PostgreSQL Tracking Table

```sql
-- Track embedding status for all entities
CREATE TABLE embedding_status (
  entity_type VARCHAR(50) NOT NULL,  -- 'member' | 'brand' | 'content' | etc.
  entity_id UUID NOT NULL,
  pinecone_id TEXT,                  -- ID in Pinecone (usually same as entity_id)
  pinecone_namespace TEXT,           -- Namespace in Pinecone
  embedding_version TEXT,            -- 'text-embedding-3-large-1536'
  last_embedded_at TIMESTAMPTZ,
  embedding_text_hash TEXT,          -- Hash of source text (detect changes)
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  PRIMARY KEY (entity_type, entity_id)
);

CREATE INDEX idx_embedding_status_last_embedded 
  ON embedding_status(last_embedded_at);

-- Trigger re-embedding if source text changes
CREATE INDEX idx_embedding_status_hash 
  ON embedding_status(embedding_text_hash);
```

### Pinecone Metadata Schema

```typescript
interface PineconeMetadata {
  // Common fields (all entities)
  entity_type: 'member' | 'brand' | 'content' | 'partnership_brief';
  entity_id: string;
  tenant_id?: string;  // For tenant-specific filtering
  indexed_at: string;  // ISO timestamp
  text_preview: string;  // First 200 chars for debugging
  
  // Entity-specific fields (brand example)
  name?: string;
  industry?: string;
  annual_revenue?: number;
  founded_year?: number;
  location?: string;
  employee_count?: number;
  
  // Searchable tags
  tags?: string[];
  categories?: string[];
}
```

---

## Performance Optimization

### Caching Strategy

```typescript
// Cache frequent semantic searches in Redis
async function cachedSemanticSearch(
  query: string,
  filters: Record<string, any>,
  ttl: number = 3600  // 1 hour default
): Promise<SearchResult[]> {
  // 1. Generate cache key (hash of query + filters)
  const cacheKey = `semantic:${hashQuery(query, filters)}`;
  
  // 2. Check cache
  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);
  
  // 3. Query Pinecone
  const results = await performSemanticSearch(query, filters);
  
  // 4. Cache results
  await redis.setex(cacheKey, ttl, JSON.stringify(results));
  
  return results;
}
```

### Batch Embedding Generation

```typescript
// Process multiple embeddings in parallel (OpenAI supports up to 2048 inputs)
async function batchEmbed(texts: string[]): Promise<number[][]> {
  const BATCH_SIZE = 100;  // Conservative batch size
  const batches = chunk(texts, BATCH_SIZE);
  
  const results = await Promise.all(
    batches.map(batch => 
      llmGateway.embed({
        model: 'text-embedding-3-large',
        input: batch,
        dimensions: 1536
      })
    )
  );
  
  return results.flatMap(r => r.data.map(d => d.embedding));
}
```

### Index Optimization

```typescript
// Use sparse-dense hybrid (Pinecone feature)
// Dense: Semantic embeddings (1536-dim)
// Sparse: Keyword boosting (BM25-style)

await index.namespace('brand').upsert([{
  id: brand.id,
  values: denseEmbedding,  // Semantic vector
  sparseValues: {
    indices: keywordIndices,  // Keyword token IDs
    values: keywordWeights    // TF-IDF weights
  },
  metadata: { ... }
}]);

// Query with hybrid scoring (70% semantic, 30% keyword)
const results = await index.namespace('brand').query({
  vector: queryEmbedding,
  sparseVector: { indices: [...], values: [...] },
  topK: 20,
  alpha: 0.7  // Weight for dense vector (0.3 for sparse)
});
```

---

## Cost Management

### Embedding Generation Costs

```typescript
// OpenAI pricing: $0.13 per 1M tokens
// text-embedding-3-large: ~8 tokens per 1000 characters

// Example cost calculation:
const AVG_TEXT_LENGTH = 500;  // characters
const TOKENS_PER_EMBED = (AVG_TEXT_LENGTH / 1000) * 8 = 4 tokens;
const COST_PER_EMBED = (4 / 1_000_000) * 0.13 = $0.00000052;

// For 100k brands × 1 embed each = $0.052
// For 1M members × 1 embed each = $0.52
// Annual updates (4x/year): ~$2.08 for 1M entities

// Budget: ~$100/month for embeddings (supports millions of updates)
```

### Pinecone Storage Costs

```typescript
// Pinecone serverless pricing (as of 2024):
// - Read units: $0.30 per 1M read units
// - Write units: $2.00 per 1M write units
// - Storage: $0.30 per GB/month

// Storage calculation:
const VECTOR_SIZE_BYTES = 1536 * 4 = 6,144 bytes (float32);
const METADATA_SIZE_BYTES = 500;  // Average metadata
const TOTAL_SIZE = 6,644 bytes per vector;

// For 1M vectors:
const STORAGE_GB = (1_000_000 * 6644) / 1_073_741_824 = 6.2 GB;
const MONTHLY_STORAGE_COST = 6.2 * 0.30 = $1.86;

// Query costs (100k queries/month):
const READ_UNITS = 100_000 * 10 = 1M read units (10 per query);
const MONTHLY_QUERY_COST = 1 * 0.30 = $0.30;

// Total Pinecone: ~$5-10/month at moderate scale
// Scales linearly with vectors and queries
```

### Cost Optimization Strategies

1. **Reduce embedding dimensions**: 3072 → 1536 (50% storage savings, minimal quality loss)
2. **Cache frequent queries**: Redis cache hot searches (reduce Pinecone queries 30-50%)
3. **Batch updates**: Update embeddings weekly/monthly vs. real-time (reduce write units)
4. **Namespace pruning**: Archive old content embeddings (reduce storage)
5. **Monitor query patterns**: Identify and cache top 20% queries (80% hit rate)

---

## Operational Considerations

### Monitoring & Alerts

**Metrics to track:**
```typescript
// Application metrics
- embedding_generation_latency_ms (p50, p95, p99)
- embedding_queue_depth (alert if >1000)
- pinecone_query_latency_ms (p95 target <200ms)
- pinecone_index_size_gb (track growth)
- embedding_pipeline_throughput (embeds/minute)
- semantic_search_cache_hit_ratio (target >60%)

// Cost metrics
- pinecone_read_units_daily
- pinecone_write_units_daily
- openai_embedding_api_calls_daily
- monthly_cost_projection

// Quality metrics
- embedding_freshness (time since last update)
- query_result_relevance_score (user feedback)
```

**Alert rules:**
```yaml
# Prometheus alerts
- alert: EmbeddingQueueBacklog
  expr: embedding_queue_depth > 5000
  for: 10m
  annotations:
    summary: "Embedding queue has >5000 pending jobs"
    
- alert: PineconeQuerySlow
  expr: histogram_quantile(0.95, pinecone_query_latency_ms) > 500
  for: 5m
  annotations:
    summary: "Pinecone p95 latency >500ms"
    
- alert: EmbeddingCostSpike
  expr: rate(openai_embedding_api_calls[1h]) > 1000
  annotations:
    summary: "Embedding API calls spiking (cost concern)"
```

### Backup & Recovery

**Pinecone backups:**
```typescript
// Pinecone doesn't support direct backups; strategy:
// 1. Maintain embedding_status table in PostgreSQL (source of truth)
// 2. Store embedding generation code/config in version control
// 3. Rebuild index from PostgreSQL if needed

async function rebuildPineconeIndex() {
  // Fetch all entities needing embeddings
  const entities = await db.selectFrom('embedding_status')
    .selectAll()
    .execute();
  
  // Re-generate and upsert in batches
  for (const batch of chunk(entities, 1000)) {
    await processEmbeddingBatch(batch);
  }
}

// Time to rebuild: ~1-2 hours for 1M vectors (with rate limits)
```

**Disaster recovery:**
- RTO: 4 hours (recreate index + re-embed)
- RPO: 24 hours (daily updates acceptable; embeddings are derived data)
- Strategy: Keep PostgreSQL as source of truth; Pinecone is cache-like

---

## Migration Path

### Phase 1: Core Embeddings (Months 1-2)
- Set up Pinecone serverless index
- Implement embedding pipeline (brands, members)
- Basic semantic search API
- Integrate with affinity service

### Phase 2: Content & Trends (Months 3-4)
- Add content embeddings
- Embed cultural trend descriptions
- Hybrid search (semantic + metadata)
- Redis caching layer

### Phase 3: Advanced Features (Months 5-6)
- Partnership brief matching
- Sparse-dense hybrid search
- Real-time embedding updates
- Quality monitoring & feedback loops

### Phase 4: Optimization (Months 7-9)
- Query performance tuning
- Cost optimization (cache tuning, batch updates)
- Reranking with cross-encoders
- Multi-modal embeddings (images, if needed)

---

## Alternatives Considered

### Alternative 1: pgvector (PostgreSQL Extension)
**Pros:** No additional service, familiar SQL, lower cost
**Cons:** Poor performance at scale (>1M vectors), manual index tuning, no serverless option
**Decision:** Viable for MVP but won't scale to TERAFFI's needs

### Alternative 2: Self-Hosted Qdrant
**Pros:** Open-source, lower cost at small scale, full control
**Cons:** Operational overhead (updates, scaling, monitoring), team lacks vector DB expertise
**Decision:** Managed Pinecone better for speed-to-market

### Alternative 3: Elasticsearch Vector Search
**Pros:** Already have experience with ES (if applicable), combines text + vector search
**Cons:** Not purpose-built for vectors, expensive at scale, complex tuning
**Decision:** Pinecone more cost-effective for vector-heavy workload

### Alternative 4: Azure Cognitive Search
**Pros:** Native Azure integration, hybrid search built-in
**Cons:** More expensive than Pinecone, less mature vector features, vendor lock-in
**Decision:** Pinecone better performance and cost

---

## Consequences

### Positive
- State-of-the-art semantic search (beyond keyword matching)
- Scalable to billions of vectors
- Fully managed (no ops overhead for vector DB)
- Fast query latency (p95 <200ms)
- Easy integration with OpenAI embeddings
- Supports hybrid search (semantic + metadata)

### Negative
- Additional service dependency (Pinecone availability)
- Eventual consistency (embedding updates are async)
- Cost scales with vector count and queries
- Learning curve for semantic search concepts
- Embeddings are "black box" (hard to debug why similarity scores are what they are)

### Neutral
- Embeddings must be regenerated if model changes (versioning strategy needed)
- Quality depends on source text quality (GIGO: garbage in, garbage out)
- Requires monitoring and tuning for optimal relevance

---

## Success Metrics

**Performance:**
- Embedding generation: <5 seconds per entity (p95)
- Semantic search: <200ms query latency (p95)
- Embedding pipeline: >1000 vectors/minute throughput
- Cache hit ratio: >60% for common queries

**Quality:**
- User feedback: >4/5 stars on search relevance
- Click-through rate: >20% on top-3 results
- Partnership match success: >70% (of suggested partnerships lead to engagement)

**Scale:**
- Support 1M+ vectors by Month 6
- Support 10M+ vectors by Month 18
- Handle 1000+ concurrent queries
- Keep monthly cost <$500 at 1M vectors

---

## References

- [Pinecone Documentation](https://docs.pinecone.io/)
- [OpenAI Embeddings Guide](https://platform.openai.com/docs/guides/embeddings)
- [Vector Search Best Practices](https://www.pinecone.io/learn/vector-search/)
- [Hybrid Search Explained](https://www.pinecone.io/learn/hybrid-search-intro/)
- [Managing Embedding Costs](https://platform.openai.com/docs/guides/embeddings/use-cases)

---
