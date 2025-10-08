# ADR-021: Search & Discovery Engine

**Status:** Proposed  
**Date:** 2025-10-01  
**Decision Makers:** Platform Architecture Team  
**Related ADRs:** ADR-011 (Neo4j), ADR-015 (Affinity Scoring), ADR-016 (Social Listening)

---

## Context

TERAFFI's core value proposition is connecting brands, IP owners, and content creators through intelligent discovery. Users need powerful search capabilities to:

**Search Use Cases:**

**Brands searching for partners:**
- "Find sustainable fashion brands targeting Gen Z"
- "Creators aligned with wellness and mindfulness"
- "IP with strong female demographics and tech affinity"

**Creators searching for opportunities:**
- "Brands looking for gaming content partnerships"
- "Companies aligned with environmental activism"
- "Deals in the esports space"

**Discovery without explicit search:**
- "Show me emerging trends I should align with"
- "Who are my best potential partners based on affinity?"
- "What partnerships are similar to my successful ones?"

**Complex queries:**
- Filter by affinity score, location, industry, values
- Sort by cultural momentum, deal value, success rate
- Faceted search with multiple dimensions

### Current Gap

Without sophisticated search:
- **Poor Discovery**: Users miss high-value partnerships
- **Manual Process**: Hours scrolling through lists
- **No Semantic Understanding**: Keyword matching misses conceptual matches
- **Stale Results**: No ranking by freshness or momentum

### Requirements

**Functional:**
1. Full-text search across entities (brands, members, content)
2. Semantic search (understand intent, not just keywords)
3. Graph-aware search (leverage relationships)
4. Faceted filtering (industry, location, values, demographics)
5. Sorting by relevance, affinity, momentum, recency
6. Autocomplete and suggestions
7. Search analytics and query understanding

**Non-Functional:**
1. Search latency <200ms (p95)
2. Handle 1000+ concurrent searches
3. Index 100k+ entities
4. Near real-time indexing (<5 minutes lag)
5. Relevance precision >80%

---

## Decision

**Implement hybrid search architecture** combining:
1. **Elasticsearch** for full-text and faceted search
2. **Vector embeddings** (OpenAI) for semantic search
3. **Neo4j** for graph-aware ranking
4. **Affinity scores** as relevance signals
5. **Multi-stage ranking** (retrieve → rerank → personalize)

**Architecture:**
```
User Query
    ↓
[Query Parser & Intent Detection]
    ↓
Parallel Search:
├─ Elasticsearch (text + filters) → Candidates
├─ Vector Search (semantic) → Candidates
└─ Neo4j (graph patterns) → Candidates
    ↓
[Fusion & Deduplication]
    ↓
[Reranking with Affinity Scores]
    ↓
[Personalization (user history)]
    ↓
Results (sorted, paginated)
```

---

## Rationale

### Why Hybrid Search

**Full-Text Search (Elasticsearch):**
- **Strengths**: Fast keyword matching, faceted filtering, fuzzy matching
- **Weaknesses**: No semantic understanding, misses synonyms/concepts
- **Use**: Initial candidate retrieval, filtering

**Semantic Search (Embeddings):**
- **Strengths**: Understands meaning, finds conceptually similar items
- **Weaknesses**: Slower, harder to explain results
- **Use**: Reranking, finding non-obvious matches

**Graph Search (Neo4j):**
- **Strengths**: Leverages relationships, finds connected entities
- **Weaknesses**: Doesn't scale for full-text
- **Use**: Relationship-based discovery, network effects

**Hybrid = Best of All:**
- Fast filtering + semantic understanding + relationship awareness
- Multiple signals improve relevance

### Technology Choices

**Elasticsearch vs Solr:**
- Both excellent, Elasticsearch has better ecosystem
- JSON-native, RESTful API, easier operations
- Better cloud offerings (Elastic Cloud)

**OpenAI Embeddings vs Sentence-BERT:**
- OpenAI: Better quality, managed service, simple API
- Sentence-BERT: Self-hosted, lower cost at scale
- Decision: Start with OpenAI, migrate to self-hosted if cost prohibitive

**Ranking Strategy:**
- Multi-stage: Fast first pass, expensive reranking on top-K
- Balances latency and quality

---

## Implementation Details

### 1. Elasticsearch Index Design

```json
// Entity index mapping
{
  "mappings": {
    "properties": {
      "id": { "type": "keyword" },
      "entity_type": { "type": "keyword" },
      "tenant_id": { "type": "keyword" },
      
      // Full-text fields
      "name": {
        "type": "text",
        "fields": {
          "keyword": { "type": "keyword" },
          "ngram": {
            "type": "text",
            "analyzer": "ngram_analyzer"
          }
        }
      },
      "description": { "type": "text" },
      "bio": { "type": "text" },
      
      // Facets
      "industry": { "type": "keyword" },
      "location": {
        "type": "geo_point"
      },
      "location_city": { "type": "keyword" },
      "location_country": { "type": "keyword" },
      "values": { "type": "keyword" },
      "demographics": { "type": "keyword" },
      "tier": { "type": "keyword" },
      
      // Numeric fields for sorting
      "affinity_score": { "type": "float" },
      "follower_count": { "type": "long" },
      "partnership_count": { "type": "integer" },
      "success_rate": { "type": "float" },
      "created_at": { "type": "date" },
      "updated_at": { "type": "date" },
      
      // Trends and momentum
      "aligned_trends": { "type": "keyword" },
      "trend_momentum_avg": { "type": "float" },
      
      // Vector embedding for semantic search
      "embedding": {
        "type": "dense_vector",
        "dims": 1536,
        "index": true,
        "similarity": "cosine"
      }
    }
  },
  "settings": {
    "analysis": {
      "analyzer": {
        "ngram_analyzer": {
          "type": "custom",
          "tokenizer": "ngram_tokenizer",
          "filter": ["lowercase"]
        }
      },
      "tokenizer": {
        "ngram_tokenizer": {
          "type": "ngram",
          "min_gram": 3,
          "max_gram": 4
        }
      }
    }
  }
}
```

### 2. Search Service

```typescript
// packages/search/src/search-service.ts

interface SearchRequest {
  query: string;
  entity_types?: string[];  // ['brand', 'member', 'content']
  filters?: {
    industry?: string[];
    location?: { lat: number; lon: number; radius: string };
    values?: string[];
    demographics?: string[];
    affinity_score?: { min?: number; max?: number };
  };
  sort?: {
    field: string;
    order: 'asc' | 'desc';
  }[];
  page?: number;
  page_size?: number;
}

interface SearchResult {
  id: string;
  entity_type: string;
  name: string;
  description: string;
  score: number;  // Combined relevance score
  affinity_score?: number;
  highlighted_fields?: Record<string, string[]>;
}

export class SearchService {
  constructor(
    private elasticsearch: ElasticsearchClient,
    private neo4j: Neo4jClient,
    private llmGateway: LLMGatewayClient
  ) {}

  async search(request: SearchRequest, userId: string): Promise<{
    results: SearchResult[];
    total: number;
    facets: Record<string, any>;
  }> {
    // 1. Parse and understand query intent
    const queryIntent = await this.parseQueryIntent(request.query);

    // 2. Build Elasticsearch query
    const esQuery = this.buildElasticsearchQuery(request, queryIntent);

    // 3. Execute parallel searches
    const [textResults, semanticResults, graphResults] = await Promise.all([
      this.searchElasticsearch(esQuery),
      this.searchSemantic(request.query, request.filters),
      this.searchGraph(request.query, userId)
    ]);

    // 4. Fusion and deduplicate
    const fusedResults = this.fuseResults([
      { results: textResults, weight: 0.4 },
      { results: semanticResults, weight: 0.3 },
      { results: graphResults, weight: 0.3 }
    ]);

    // 5. Rerank top candidates with affinity scores
    const reranked = await this.rerankWithAffinity(
      fusedResults.slice(0, 100),  // Top 100 candidates
      userId
    );

    // 6. Apply user personalization
    const personalized = await this.personalizeResults(reranked, userId);

    // 7. Paginate
    const pageSize = request.page_size || 20;
    const page = request.page || 1;
    const start = (page - 1) * pageSize;
    const results = personalized.slice(start, start + pageSize);

    // 8. Extract facets
    const facets = this.extractFacets(textResults);

    return {
      results,
      total: fusedResults.length,
      facets
    };
  }

  private async parseQueryIntent(query: string): Promise<{
    keywords: string[];
    intent: 'find_partners' | 'discover_trends' | 'explore_similar' | 'general';
    filters: any;
  }> {
    // Use LLM to understand query intent
    const response = await this.llmGateway.complete({
      model: 'gpt-4o-mini',
      messages: [{
        role: 'system',
        content: `Parse search query intent. Extract:
- keywords: important search terms
- intent: type of search (find_partners, discover_trends, explore_similar, general)
- implied_filters: filters mentioned in query (e.g., "Gen Z" → demographics filter)

Return JSON.`
      }, {
        role: 'user',
        content: query
      }],
      response_format: { type: 'json_object' }
    });

    return JSON.parse(response.choices[0].message.content);
  }

  private buildElasticsearchQuery(request: SearchRequest, intent: any): any {
    const must: any[] = [];
    const filter: any[] = [];

    // Tenant isolation
    filter.push({
      term: { tenant_id: this.getTenantId() }
    });

    // Entity type filter
    if (request.entity_types?.length) {
      filter.push({
        terms: { entity_type: request.entity_types }
      });
    }

    // Main query
    if (request.query) {
      must.push({
        multi_match: {
          query: request.query,
          fields: [
            'name^3',           // Boost name matches
            'description^2',
            'bio',
            'values',
            'aligned_trends'
          ],
          type: 'best_fields',
          fuzziness: 'AUTO'
        }
      });
    }

    // Apply filters
    if (request.filters) {
      if (request.filters.industry?.length) {
        filter.push({ terms: { industry: request.filters.industry } });
      }

      if (request.filters.location) {
        filter.push({
          geo_distance: {
            distance: request.filters.location.radius,
            location: {
              lat: request.filters.location.lat,
              lon: request.filters.location.lon
            }
          }
        });
      }

      if (request.filters.values?.length) {
        filter.push({ terms: { values: request.filters.values } });
      }

      if (request.filters.demographics?.length) {
        filter.push({ terms: { demographics: request.filters.demographics } });
      }

      if (request.filters.affinity_score) {
        filter.push({
          range: {
            affinity_score: {
              gte: request.filters.affinity_score.min,
              lte: request.filters.affinity_score.max
            }
          }
        });
      }
    }

    return {
      query: {
        bool: { must, filter }
      },
      sort: this.buildSortCriteria(request.sort),
      highlight: {
        fields: {
          name: {},
          description: {},
          bio: {}
        }
      },
      aggs: {
        industries: { terms: { field: 'industry', size: 20 } },
        values: { terms: { field: 'values', size: 20 } },
        demographics: { terms: { field: 'demographics', size: 20 } }
      }
    };
  }

  private async searchElasticsearch(query: any): Promise<SearchResult[]> {
    const response = await this.elasticsearch.search({
      index: 'entities',
      body: query,
      size: 100
    });

    return response.hits.hits.map(hit => ({
      id: hit._source.id,
      entity_type: hit._source.entity_type,
      name: hit._source.name,
      description: hit._source.description,
      score: hit._score,
      affinity_score: hit._source.affinity_score,
      highlighted_fields: hit.highlight
    }));
  }

  private async searchSemantic(
    query: string,
    filters?: any
  ): Promise<SearchResult[]> {
    // 1. Generate embedding for query
    const queryEmbedding = await this.generateEmbedding(query);

    // 2. Vector search in Elasticsearch
    const response = await this.elasticsearch.search({
      index: 'entities',
      body: {
        query: {
          bool: {
            must: [{
              script_score: {
                query: { match_all: {} },
                script: {
                  source: "cosineSimilarity(params.query_vector, 'embedding') + 1.0",
                  params: { query_vector: queryEmbedding }
                }
              }
            }],
            filter: this.buildFilters(filters)
          }
        },
        size: 50
      }
    });

    return response.hits.hits.map(hit => ({
      id: hit._source.id,
      entity_type: hit._source.entity_type,
      name: hit._source.name,
      description: hit._source.description,
      score: hit._score,
      affinity_score: hit._source.affinity_score
    }));
  }

  private async searchGraph(query: string, userId: string): Promise<SearchResult[]> {
    // Graph-based search: find entities connected to user's network
    const session = this.neo4j.session();

    try {
      const result = await session.run(`
        MATCH (user:Member {id: $userId})
        MATCH (user)-[:PARTNERS_WITH*1..2]-(connected)
        WHERE connected.name CONTAINS $query
          OR any(value IN connected.values WHERE value CONTAINS $query)
        WITH connected, 
             length((user)-[:PARTNERS_WITH*]-(connected)) as distance
        RETURN connected.id as id,
               connected.entity_type as entity_type,
               connected.name as name,
               connected.description as description,
               1.0 / (distance + 1) as score
        ORDER BY score DESC
        LIMIT 50
      `, {
        userId,
        query
      });

      return result.records.map(record => ({
        id: record.get('id'),
        entity_type: record.get('entity_type'),
        name: record.get('name'),
        description: record.get('description'),
        score: record.get('score')
      }));
    } finally {
      await session.close();
    }
  }

  private fuseResults(sources: Array<{
    results: SearchResult[];
    weight: number;
  }>): SearchResult[] {
    // Reciprocal Rank Fusion algorithm
    const fusedScores = new Map<string, number>();

    for (const source of sources) {
      source.results.forEach((result, rank) => {
        const rrfScore = source.weight / (rank + 60);  // RRF constant = 60
        const currentScore = fusedScores.get(result.id) || 0;
        fusedScores.set(result.id, currentScore + rrfScore);
      });
    }

    // Create deduplicated result set
    const resultMap = new Map<string, SearchResult>();
    for (const source of sources) {
      for (const result of source.results) {
        if (!resultMap.has(result.id)) {
          resultMap.set(result.id, result);
        }
      }
    }

    // Sort by fused score
    return Array.from(resultMap.values())
      .map(result => ({
        ...result,
        score: fusedScores.get(result.id) || 0
      }))
      .sort((a, b) => b.score - a.score);
  }

  private async rerankWithAffinity(
    candidates: SearchResult[],
    userId: string
  ): Promise<SearchResult[]> {
    // Load affinity scores for user → candidates
    const candidateIds = candidates.map(c => c.id);
    
    const affinities = await this.db
      .selectFrom('affinity_scores')
      .select(['entity2_id', 'affinity_score'])
      .where('entity1_id', '=', userId)
      .where('entity2_id', 'in', candidateIds)
      .execute();

    const affinityMap = new Map(
      affinities.map(a => [a.entity2_id, a.affinity_score])
    );

    // Boost scores with affinity
    return candidates.map(result => ({
      ...result,
      affinity_score: affinityMap.get(result.id) || 0,
      score: result.score * 0.6 + (affinityMap.get(result.id) || 0) * 0.4
    })).sort((a, b) => b.score - a.score);
  }

  private async personalizeResults(
    results: SearchResult[],
    userId: string
  ): Promise<SearchResult[]> {
    // Personalize based on user's search history and preferences
    const userHistory = await this.getUserSearchHistory(userId);

    // Boost results similar to previously clicked items
    // Penalize results user has already seen many times
    // This is a simplified example
    return results;
  }

  private async generateEmbedding(text: string): Promise<number[]> {
    const response = await this.llmGateway.embeddings({
      model: 'text-embedding-3-small',
      input: text
    });

    return response.data[0].embedding;
  }
}
```

### 3. Indexing Pipeline

```typescript
// packages/search/src/indexer.ts

export class SearchIndexer {
  async indexEntity(entity: Entity): Promise<void> {
    // 1. Generate embedding for entity
    const textToEmbed = this.buildEmbeddingText(entity);
    const embedding = await this.generateEmbedding(textToEmbed);

    // 2. Enrich with graph data
    const graphData = await this.getGraphData(entity.id);

    // 3. Build search document
    const document = {
      id: entity.id,
      entity_type: entity.type,
      tenant_id: entity.tenant_id,
      name: entity.name,
      description: entity.description,
      bio: entity.bio,
      industry: entity.industry,
      location: entity.location,
      location_city: entity.location_city,
      location_country: entity.location_country,
      values: entity.values,
      demographics: graphData.target_demographics,
      tier: entity.tier,
      affinity_score: graphData.avg_affinity_score,
      follower_count: entity.follower_count,
      partnership_count: graphData.partnership_count,
      success_rate: graphData.success_rate,
      aligned_trends: graphData.aligned_trends,
      trend_momentum_avg: graphData.trend_momentum_avg,
      embedding,
      created_at: entity.created_at,
      updated_at: entity.updated_at
    };

    // 4. Index in Elasticsearch
    await this.elasticsearch.index({
      index: 'entities',
      id: entity.id,
      body: document
    });
  }

  async reindexAll(): Promise<void> {
    // Bulk reindex all entities
    const entities = await this.db
      .selectFrom('entities')
      .selectAll()
      .execute();

    const batchSize = 100;
    for (let i = 0; i < entities.length; i += batchSize) {
      const batch = entities.slice(i, i + batchSize);
      await Promise.all(batch.map(e => this.indexEntity(e)));
    }
  }

  private buildEmbeddingText(entity: Entity): string {
    // Create rich text representation for embedding
    const parts = [
      entity.name,
      entity.description,
      entity.bio,
      `Industry: ${entity.industry}`,
      `Values: ${entity.values?.join(', ')}`,
      `Location: ${entity.location_city}, ${entity.location_country}`
    ].filter(Boolean);

    return parts.join(' | ');
  }

  private async getGraphData(entityId: string): Promise<any> {
    const session = this.neo4j.session();

    try {
      const result = await session.run(`
        MATCH (e {id: $entityId})
        OPTIONAL MATCH (e)-[:PARTNERS_WITH]-(partner)
        OPTIONAL MATCH (e)-[:ALIGNS_WITH]->(trend:Trend)
        OPTIONAL MATCH (e)-[:TARGETS]->(demo:Demographic)
        RETURN
          count(DISTINCT partner) as partnership_count,
          collect(DISTINCT demo.segment) as target_demographics,
          collect(DISTINCT trend.name) as aligned_trends,
          avg(trend.momentum_score) as trend_momentum_avg
      `, { entityId });

      return result.records[0]?.toObject() || {};
    } finally {
      await session.close();
    }
  }
}
```

### 4. Autocomplete Service

```typescript
// packages/search/src/autocomplete.ts

export class AutocompleteService {
  async suggest(prefix: string, limit: number = 10): Promise<Array<{
    text: string;
    type: 'entity' | 'trend' | 'industry' | 'value';
    id?: string;
  }>> {
    const response = await this.elasticsearch.search({
      index: 'entities',
      body: {
        query: {
          bool: {
            must: [{
              multi_match: {
                query: prefix,
                fields: ['name.ngram', 'name'],
                type: 'bool_prefix'
              }
            }],
            filter: [{
              term: { tenant_id: this.getTenantId() }
            }]
          }
        },
        size: limit,
        _source: ['id', 'name', 'entity_type']
      }
    });

    const entitySuggestions = response.hits.hits.map(hit => ({
      text: hit._source.name,
      type: 'entity' as const,
      id: hit._source.id
    }));

    // Also suggest trends
    const trendResponse = await this.elasticsearch.search({
      index: 'trends',
      body: {
        query: {
          match: {
            name: {
              query: prefix,
              fuzziness: 'AUTO'
            }
          }
        },
        size: 5
      }
    });

    const trendSuggestions = trendResponse.hits.hits.map(hit => ({
      text: hit._source.name,
      type: 'trend' as const
    }));

    return [...entitySuggestions, ...trendSuggestions].slice(0, limit);
  }
}
```

---

## Database Schema Updates

```sql
-- Search analytics
CREATE TABLE search_queries (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  tenant_id UUID REFERENCES tenants(id),
  query TEXT NOT NULL,
  filters JSONB,
  results_count INTEGER,
  clicked_result_ids UUID[],
  timestamp TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_search_queries_user ON search_queries(user_id, timestamp DESC);
CREATE INDEX idx_search_queries_query ON search_queries USING gin(to_tsvector('english', query));

-- Popular searches
CREATE MATERIALIZED VIEW popular_searches AS
SELECT
  tenant_id,
  query,
  COUNT(*) as search_count,
  COUNT(DISTINCT user_id) as unique_users
FROM search_queries
WHERE timestamp > NOW() - INTERVAL '30 days'
GROUP BY tenant_id, query
HAVING COUNT(*) > 5
ORDER BY search_count DESC;

CREATE INDEX idx_popular_searches_tenant ON popular_searches(tenant_id);
```

---

## Migration Path

### Phase 1: Foundation (Months 1-2)
- Elasticsearch cluster setup
- Basic full-text search
- Index pipeline
- Autocomplete

### Phase 2: Semantic Search (Months 3-4)
- Embedding generation
- Vector search
- Query intent parsing

### Phase 3: Graph Integration (Months 5-6)
- Neo4j search integration
- Result fusion
- Affinity-based reranking

### Phase 4: Personalization (Months 7-8)
- Search history tracking
- Personalized ranking
- Search analytics dashboard

---

## Success Metrics

**Performance:**
- Search latency <200ms (p95)
- Autocomplete <50ms (p95)
- Index refresh <5 minutes

**Relevance:**
- Click-through rate >40%
- Zero-result searches <5%
- User satisfaction score >4.0/5

**Scale:**
- Handle 1000+ QPS
- 100k+ entities indexed
- Support 10k+ concurrent users

---

## Task Dependencies

001 (Elasticsearch) → 002 (Index Design) → 003 (Embeddings) → 004 (Indexer)
                                        → 005 (Full-Text) ───┐
                                        → 006 (Semantic) ────┤
                                                             ├→ 009 (Fusion) → 010 (Reranking) → 011 (Personalization) → 012 (Main Service)
002 → 007 (Graph Search) ───────────────────────────────────┘
005 → 008 (Intent Parser)

002 → 013 (Autocomplete)
012 → 014 (Analytics)
012 → 015 (REST API)

004 → 016 (Real-time CDC)
015 → 017 (Performance)
005 → 018 (Facets)
015 → 019 (Monitoring)

All → 020 (Test Suite)
All → 021 (Documentation)

## References

- [Elasticsearch Guide](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
- [Vector Search Best Practices](https://www.pinecone.io/learn/vector-search/)
- [Reciprocal Rank Fusion](https://plg.uwaterloo.ca/~gvcormac/cormacksigir09-rrf.pdf)
- [Learning to Rank](https://en.wikipedia.org/wiki/Learning_to_rank)

---
