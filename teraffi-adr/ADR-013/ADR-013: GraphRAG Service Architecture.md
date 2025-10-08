# ADR-013: GraphRAG Service Architecture

**Status:** Proposed  
**Date:** 2025-10-01  
**Decision Makers:** Platform Architecture Team  
**Related ADRs:** ADR-011 (Neo4j), ADR-012 (Vector Embeddings), ADR-014 (LLM Gateway)

---

## Context

TERAFFI's Intelligence Pillar requires answering complex questions that combine:
- **Structured relationships** (partnerships, org hierarchies, content networks)
- **Semantic understanding** (brand values, cultural trends, member goals)
- **Temporal patterns** (trend evolution, partnership history)
- **Multi-hop reasoning** ("Find brands aligned with creators who influence demographics trending toward sustainability")

Traditional approaches fail:

**Pure Graph Queries (Neo4j alone):**
- Cannot understand semantic nuance ("eco-friendly" vs "sustainable" vs "regenerative")
- Require explicit relationships (missing implicit connections)
- Struggle with natural language queries

**Pure Vector Search (Pinecone alone):**
- No understanding of relationships (who works with whom, influence chains)
- Cannot traverse multi-hop paths
- Miss structural patterns (community clusters, bottleneck nodes)

**Pure LLM Queries:**
- Hallucinate facts not in knowledge base
- Cannot reason over large, dynamic relationship graphs
- Expensive at scale (full context in every prompt)

**GraphRAG Solution:**
Combine graph structure (Neo4j) + semantic embeddings (Pinecone) + LLM reasoning to answer questions like:
- "Recommend partnership opportunities for Brand X based on successful similar collaborations"
- "Which cultural trends are emerging among creators aligned with our values?"
- "Find potential partners in the sustainability space with strong community engagement"

### Requirements

**Functional:**
1. Accept natural language queries about partnerships, affinities, trends
2. Retrieve relevant subgraph from Neo4j (entities + relationships)
3. Retrieve semantic matches from Pinecone (similar entities)
4. Synthesize graph + vector context into LLM prompt
5. Generate natural language responses with citations
6. Support follow-up questions (conversational context)

**Non-Functional:**
1. Query latency p95 < 5 seconds (complex multi-hop queries)
2. Accuracy: answers grounded in actual data (no hallucinations)
3. Cost: <$0.10 per query (LLM + infrastructure)
4. Scalability: 100+ concurrent queries
5. Explainability: show reasoning path (which entities/relationships informed answer)

---

## Decision

**Implement GraphRAG as a service layer** that orchestrates Neo4j, Pinecone, and LLM Gateway to answer complex queries over TERAFFI's knowledge graph.

**Architecture:**
```
User Query (Natural Language)
    ↓
1. Query Understanding (LLM)
   - Extract entities, intent, constraints
    ↓
2. Graph Retrieval (Neo4j)
   - Fetch relevant subgraph (2-3 hops)
    ↓
3. Semantic Augmentation (Pinecone)
   - Find semantically similar entities
    ↓
4. Context Assembly
   - Combine graph + embeddings into structured context
    ↓
5. Answer Generation (LLM)
   - Synthesize answer with citations
    ↓
Response (Natural Language + Evidence)
```

**Technology Stack:**
- **Neo4j** (via ADR-011): Relationship graph traversal
- **Pinecone** (via ADR-012): Semantic similarity search
- **LLM Gateway** (via ADR-014): Query understanding + answer synthesis
- **Redis** (port 6380): Query cache, conversation context
- **PostgreSQL**: Query logging, feedback storage

---

## Rationale

### Why GraphRAG Over Alternatives

**GraphRAG vs. Pure RAG (Retrieval-Augmented Generation):**
- RAG: Retrieve documents → embed → vector search → LLM synthesis
- GraphRAG: Retrieve graph + vectors → reason over relationships → LLM synthesis
- **Advantage**: GraphRAG understands connections, not just content similarity
- **Use case**: "Find partners who have succeeded with similar brands" requires relationship reasoning, not just semantic similarity

**GraphRAG vs. Pure Knowledge Graph:**
- Pure KG: Structured queries only (Cypher), no natural language
- GraphRAG: Natural language interface + semantic understanding
- **Advantage**: Accessible to non-technical users, handles ambiguous queries
- **Use case**: Business users ask "Who should we partner with?" not write Cypher queries

**GraphRAG vs. LangChain Agents:**
- LangChain: Generic agent framework, multiple tool calls
- GraphRAG: Specialized pipeline for graph + vector + LLM
- **Advantage**: Optimized for graph reasoning, predictable costs, faster
- **Use case**: TERAFFI's queries are graph-centric; generic agents add overhead

**GraphRAG vs. Microsoft GraphRAG:**
- Microsoft: Document-centric (extract graphs from docs)
- TERAFFI: Database-centric (query existing graph)
- **Difference**: We have structured graph (Neo4j); Microsoft extracts from unstructured text
- **Decision**: Custom implementation fits our use case better

---

## Implementation Details

### 1. Query Understanding (Intent Extraction)

```typescript
// packages/graphrag-service/src/query-understanding.ts

interface QueryIntent {
  primary_action: 'find' | 'recommend' | 'analyze' | 'compare';
  entity_types: string[];  // ['brand', 'member', 'content']
  constraints: QueryConstraint[];
  semantic_query: string;  // Reformulated for embedding
}

interface QueryConstraint {
  type: 'filter' | 'relation' | 'temporal';
  field: string;
  operator: string;
  value: any;
}

export class QueryUnderstanding {
  constructor(private llmGateway: LLMGatewayClient) {}

  async extractIntent(query: string): Promise<QueryIntent> {
    // Use LLM to parse natural language into structured intent
    const response = await this.llmGateway.complete({
      model: 'gpt-4o-mini',  // Fast, cheap model for parsing
      messages: [{
        role: 'system',
        content: `You are a query parser for a partnership intelligence platform.
Extract structured intent from natural language queries.
Output JSON with: primary_action, entity_types, constraints, semantic_query.

Example:
Query: "Find sustainable fashion brands in Europe with >$10M revenue"
Output: {
  "primary_action": "find",
  "entity_types": ["brand"],
  "constraints": [
    {"type": "filter", "field": "industry", "operator": "contains", "value": "fashion"},
    {"type": "filter", "field": "location", "operator": "in", "value": "Europe"},
    {"type": "filter", "field": "annual_revenue", "operator": ">=", "value": 10000000}
  ],
  "semantic_query": "sustainable fashion brands focus on eco-friendly practices"
}`
      }, {
        role: 'user',
        content: query
      }],
      response_format: { type: 'json_object' },
      temperature: 0.0
    });

    return JSON.parse(response.choices[0].message.content);
  }
}
```

### 2. Graph Retrieval (Neo4j Subgraph Extraction)

```typescript
// packages/graphrag-service/src/graph-retrieval.ts

interface GraphContext {
  nodes: GraphNode[];
  relationships: GraphRelationship[];
  metadata: {
    query_time_ms: number;
    node_count: number;
    relationship_count: number;
  };
}

export class GraphRetrieval {
  constructor(private neo4jClient: Neo4jClient) {}

  async retrieveSubgraph(intent: QueryIntent): Promise<GraphContext> {
    // Build Cypher query based on intent
    const cypher = this.buildCypherFromIntent(intent);
    
    const session = this.neo4jClient.session();
    try {
      const startTime = Date.now();
      const result = await session.run(cypher, intent.constraints);
      
      // Extract nodes and relationships from result
      const nodes = this.extractNodes(result);
      const relationships = this.extractRelationships(result);
      
      return {
        nodes,
        relationships,
        metadata: {
          query_time_ms: Date.now() - startTime,
          node_count: nodes.length,
          relationship_count: relationships.length
        }
      };
    } finally {
      await session.close();
    }
  }

  private buildCypherFromIntent(intent: QueryIntent): string {
    // Example: Find brands with specific attributes + their partnerships
    if (intent.primary_action === 'find' && intent.entity_types.includes('brand')) {
      return `
        MATCH (b:Brand)
        WHERE b.industry CONTAINS $industry_filter
          AND b.annual_revenue >= $min_revenue
        OPTIONAL MATCH (b)-[r:PARTNERS_WITH]-(partner:Member)
        OPTIONAL MATCH (b)-[align:ALIGNS_WITH]->(t:Trend)
        RETURN b, r, partner, align, t
        LIMIT 50
      `;
    }
    
    // Add more query patterns for different intents
    // ...
    
    throw new Error(`Unsupported intent: ${intent.primary_action}`);
  }
}
```

### 3. Semantic Augmentation (Pinecone)

```typescript
// packages/graphrag-service/src/semantic-augmentation.ts

interface SemanticContext {
  similar_entities: Array<{
    id: string;
    type: string;
    similarity_score: number;
    metadata: Record<string, any>;
  }>;
  metadata: {
    query_time_ms: number;
    result_count: number;
  };
}

export class SemanticAugmentation {
  constructor(
    private llmGateway: LLMGatewayClient,
    private pineconeClient: PineconeClient
  ) {}

  async augmentWithSemantics(
    intent: QueryIntent,
    graphContext: GraphContext
  ): Promise<SemanticContext> {
    // Generate embedding for semantic query
    const embedding = await this.llmGateway.embed({
      model: 'text-embedding-3-large',
      input: intent.semantic_query,
      dimensions: 1536
    });

    // Query Pinecone for semantically similar entities
    const startTime = Date.now();
    const index = this.pineconeClient.Index('teraffi-prod');
    
    // Query relevant namespaces based on entity types
    const namespaces = intent.entity_types.map(type => 
      type === 'member' ? 'member' : 
      type === 'brand' ? 'brand' : 
      type === 'content' ? 'content' : 'trend'
    );

    const results = await Promise.all(
      namespaces.map(ns => 
        index.namespace(ns).query({
          vector: embedding.data[0].embedding,
          topK: 20,
          includeMetadata: true,
          filter: this.buildPineconeFilter(intent.constraints)
        })
      )
    );

    // Flatten and deduplicate results
    const similar_entities = results
      .flatMap(r => r.matches)
      .map(match => ({
        id: match.id,
        type: this.inferTypeFromMetadata(match.metadata),
        similarity_score: match.score,
        metadata: match.metadata
      }))
      .sort((a, b) => b.similarity_score - a.similarity_score)
      .slice(0, 30);  // Top 30 across all namespaces

    return {
      similar_entities,
      metadata: {
        query_time_ms: Date.now() - startTime,
        result_count: similar_entities.length
      }
    };
  }
}
```

### 4. Context Assembly

```typescript
// packages/graphrag-service/src/context-assembly.ts

interface AssembledContext {
  structured_context: string;  // Formatted for LLM
  sources: Source[];           // For citations
  token_count: number;
}

interface Source {
  id: string;
  type: 'graph_node' | 'graph_relationship' | 'semantic_match';
  content: string;
  metadata: Record<string, any>;
}

export class ContextAssembly {
  async assemble(
    query: string,
    graphContext: GraphContext,
    semanticContext: SemanticContext
  ): Promise<AssembledContext> {
    const sources: Source[] = [];
    
    // 1. Format graph nodes
    const nodeDescriptions = graphContext.nodes.map((node, idx) => {
      const source: Source = {
        id: `node_${idx}`,
        type: 'graph_node',
        content: this.formatNode(node),
        metadata: node.properties
      };
      sources.push(source);
      return `[${source.id}] ${source.content}`;
    }).join('\n');

    // 2. Format relationships
    const relationshipDescriptions = graphContext.relationships.map((rel, idx) => {
      const source: Source = {
        id: `rel_${idx}`,
        type: 'graph_relationship',
        content: this.formatRelationship(rel),
        metadata: rel.properties
      };
      sources.push(source);
      return `[${source.id}] ${source.content}`;
    }).join('\n');

    // 3. Format semantic matches
    const semanticDescriptions = semanticContext.similar_entities.map((entity, idx) => {
      const source: Source = {
        id: `sem_${idx}`,
        type: 'semantic_match',
        content: this.formatSemanticMatch(entity),
        metadata: entity.metadata
      };
      sources.push(source);
      return `[${source.id}] ${source.content}`;
    }).join('\n');

    // 4. Assemble structured context
    const structured_context = `
# Knowledge Graph Context

## Entities
${nodeDescriptions}

## Relationships
${relationshipDescriptions}

## Semantically Similar Entities
${semanticDescriptions}

# Query
${query}

Please answer the query using the context above. Cite sources using [source_id] notation.
    `.trim();

    // 5. Estimate token count (rough)
    const token_count = Math.ceil(structured_context.length / 4);

    return {
      structured_context,
      sources,
      token_count
    };
  }

  private formatNode(node: GraphNode): string {
    // Example: "Brand 'EcoWear' (sustainable_fashion) - $500M revenue, 1500 employees"
    return `${node.labels[0]} '${node.properties.name}' (${node.properties.industry}) - ${this.summarizeProperties(node.properties)}`;
  }

  private formatRelationship(rel: GraphRelationship): string {
    // Example: "'Sarah Bennett' PARTNERS_WITH 'EcoWear' (status: active, value: $250k, success_score: 4.3)"
    return `'${rel.start_node_name}' ${rel.type} '${rel.end_node_name}' (${this.summarizeProperties(rel.properties)})`;
  }

  private formatSemanticMatch(entity: any): string {
    // Example: "Brand 'Patagonia' (similarity: 0.89) - outdoor apparel, environmental activism"
    return `${entity.type} '${entity.metadata.name}' (similarity: ${entity.similarity_score.toFixed(2)}) - ${entity.metadata.description || 'no description'}`;
  }
}
```

### 5. Answer Generation (LLM Synthesis)

```typescript
// packages/graphrag-service/src/answer-generation.ts

interface GraphRAGAnswer {
  answer: string;
  citations: Citation[];
  confidence: number;
  reasoning_path: string[];
  metadata: {
    llm_model: string;
    tokens_used: number;
    query_time_ms: number;
  };
}

interface Citation {
  source_id: string;
  type: 'graph_node' | 'graph_relationship' | 'semantic_match';
  content: string;
}

export class AnswerGeneration {
  constructor(private llmGateway: LLMGatewayClient) {}

  async generateAnswer(
    query: string,
    context: AssembledContext
  ): Promise<GraphRAGAnswer> {
    const startTime = Date.now();
    
    const response = await this.llmGateway.complete({
      model: 'gpt-4o',  // Best reasoning model
      messages: [{
        role: 'system',
        content: `You are an expert partnership intelligence analyst.
Answer questions based ONLY on the provided knowledge graph context.
Always cite sources using [source_id] notation.
If information is not in the context, say so - do not make up information.
Provide reasoning for your conclusions.`
      }, {
        role: 'user',
        content: context.structured_context
      }],
      temperature: 0.3,  // Low temperature for factual answers
      max_tokens: 1000
    });

    const answer_text = response.choices[0].message.content;
    
    // Extract citations from answer
    const citations = this.extractCitations(answer_text, context.sources);
    
    // Estimate confidence based on citation count and semantic scores
    const confidence = this.estimateConfidence(citations, context);

    return {
      answer: answer_text,
      citations,
      confidence,
      reasoning_path: this.extractReasoningPath(answer_text),
      metadata: {
        llm_model: 'gpt-4o',
        tokens_used: response.usage.total_tokens,
        query_time_ms: Date.now() - startTime
      }
    };
  }

  private extractCitations(answer: string, sources: Source[]): Citation[] {
    // Find all [source_id] references in answer
    const citationRegex = /\[(node|rel|sem)_\d+\]/g;
    const matches = answer.match(citationRegex) || [];
    
    return matches.map(match => {
      const sourceId = match.slice(1, -1);  // Remove brackets
      const source = sources.find(s => s.id === sourceId);
      return {
        source_id: sourceId,
        type: source?.type || 'unknown',
        content: source?.content || 'Source not found'
      };
    });
  }

  private estimateConfidence(citations: Citation[], context: AssembledContext): number {
    // Simple heuristic: more citations + higher semantic scores = higher confidence
    const citationCount = citations.length;
    const maxCitations = 10;
    const citationScore = Math.min(citationCount / maxCitations, 1.0);
    
    // Average semantic similarity of cited sources
    const semanticCitations = citations.filter(c => c.type === 'semantic_match');
    const avgSemanticScore = semanticCitations.length > 0
      ? semanticCitations.reduce((sum, c) => {
          const source = context.sources.find(s => s.id === c.source_id);
          return sum + (source?.metadata?.similarity_score || 0);
        }, 0) / semanticCitations.length
      : 0.5;
    
    // Weighted average
    return (citationScore * 0.4 + avgSemanticScore * 0.6);
  }
}
```

### 6. Orchestration Service

```typescript
// packages/graphrag-service/src/graphrag-service.ts

export class GraphRAGService {
  constructor(
    private queryUnderstanding: QueryUnderstanding,
    private graphRetrieval: GraphRetrieval,
    private semanticAugmentation: SemanticAugmentation,
    private contextAssembly: ContextAssembly,
    private answerGeneration: AnswerGeneration,
    private cache: RedisCache
  ) {}

  async query(request: GraphRAGQueryRequest): Promise<GraphRAGAnswer> {
    // 1. Check cache
    const cacheKey = `graphrag:${hashQuery(request.query)}`;
    const cached = await this.cache.get(cacheKey);
    if (cached && !request.bypass_cache) {
      return JSON.parse(cached);
    }

    // 2. Understand query intent
    const intent = await this.queryUnderstanding.extractIntent(request.query);

    // 3. Retrieve graph context
    const graphContext = await this.graphRetrieval.retrieveSubgraph(intent);

    // 4. Augment with semantic matches
    const semanticContext = await this.semanticAugmentation.augmentWithSemantics(
      intent,
      graphContext
    );

    // 5. Assemble context
    const assembledContext = await this.contextAssembly.assemble(
      request.query,
      graphContext,
      semanticContext
    );

    // 6. Generate answer
    const answer = await this.answerGeneration.generateAnswer(
      request.query,
      assembledContext
    );

    // 7. Cache result
    await this.cache.setex(cacheKey, 3600, JSON.stringify(answer));  // 1 hour TTL

    // 8. Log query for analytics
    await this.logQuery(request, answer);

    return answer;
  }

  private async logQuery(
    request: GraphRAGQueryRequest,
    answer: GraphRAGAnswer
  ): Promise<void> {
    await this.db.insertInto('graphrag_queries')
      .values({
        query_text: request.query,
        user_id: request.user_id,
        confidence: answer.confidence,
        tokens_used: answer.metadata.tokens_used,
        query_time_ms: answer.metadata.query_time_ms,
        created_at: new Date()
      })
      .execute();
  }
}
```

---

## Conversational Context (Follow-up Questions)

```typescript
// packages/graphrag-service/src/conversation-manager.ts

interface ConversationContext {
  conversation_id: string;
  messages: ConversationMessage[];
  graph_context: GraphContext;  // Reuse from previous query
  created_at: Date;
  expires_at: Date;
}

interface ConversationMessage {
  role: 'user' | 'assistant';
  content: string;
  timestamp: Date;
}

export class ConversationManager {
  constructor(private redis: RedisCache) {}

  async createConversation(): Promise<string> {
    const conversation_id = generateId('conv_');
    const context: ConversationContext = {
      conversation_id,
      messages: [],
      graph_context: { nodes: [], relationships: [], metadata: {} },
      created_at: new Date(),
      expires_at: new Date(Date.now() + 3600000)  // 1 hour
    };
    
    await this.redis.setex(
      `conversation:${conversation_id}`,
      3600,
      JSON.stringify(context)
    );
    
    return conversation_id;
  }

  async addMessage(
    conversation_id: string,
    role: 'user' | 'assistant',
    content: string
  ): Promise<void> {
    const context = await this.getConversation(conversation_id);
    context.messages.push({
      role,
      content,
      timestamp: new Date()
    });
    
    await this.redis.setex(
      `conversation:${conversation_id}`,
      3600,
      JSON.stringify(context)
    );
  }

  async getConversation(conversation_id: string): Promise<ConversationContext> {
    const cached = await this.redis.get(`conversation:${conversation_id}`);
    if (!cached) {
      throw new Error('Conversation not found or expired');
    }
    return JSON.parse(cached);
  }

  // Include conversation history in LLM prompt for follow-ups
  formatConversationHistory(context: ConversationContext): string {
    return context.messages
      .map(m => `${m.role === 'user' ? 'User' : 'Assistant'}: ${m.content}`)
      .join('\n\n');
  }
}
```

---

## Performance Optimization

### Caching Strategy

```typescript
// Multi-level caching

// Level 1: Full query results (1 hour TTL)
const queryCache = `graphrag:${hashQuery(query)}`;

// Level 2: Graph context (6 hours TTL - reuse for similar queries)
const graphCache = `graph_context:${hashIntent(intent)}`;

// Level 3: Semantic results (12 hours TTL - embeddings change slowly)
const semanticCache = `semantic:${hashSemanticQuery(intent.semantic_query)}`;
```

### Parallel Execution

```typescript
// Execute graph and semantic retrieval in parallel
const [graphContext, semanticContext] = await Promise.all([
  this.graphRetrieval.retrieveSubgraph(intent),
  this.semanticAugmentation.augmentWithSemantics(intent, {} as GraphContext)
]);
```

### Context Window Management

```typescript
// Limit context size to fit in LLM context window
const MAX_CONTEXT_TOKENS = 8000;  // Reserve tokens for answer

function truncateContext(context: AssembledContext): AssembledContext {
  if (context.token_count <= MAX_CONTEXT_TOKENS) {
    return context;
  }
  
  // Prioritize: high-scoring semantic matches > direct relationships > distant nodes
  const sources = [
    ...context.sources.filter(s => s.type === 'semantic_match' && s.metadata.similarity_score > 0.8),
    ...context.sources.filter(s => s.type === 'graph_relationship'),
    ...context.sources.filter(s => s.type === 'graph_node')
  ].slice(0, MAX_CONTEXT_TOKENS / 10);  // Rough estimate: 10 tokens per source

  return rebuildContext(sources);
}
```

---

## Cost Management

```typescript
// Estimated costs per query:
// - Query understanding (GPT-4o-mini): ~500 tokens × $0.15/1M = $0.00008
// - Graph retrieval (Neo4j): negligible (self-hosted)
// - Semantic search (Pinecone): ~10 read units × $0.30/1M = $0.000003
// - Answer generation (GPT-4o): ~2000 tokens × $5/1M = $0.01
// Total: ~$0.01 per query

// Cost optimization strategies:
// 1. Cache aggressively (60%+ cache hit rate)
// 2. Use GPT-4o-mini for simple queries (classification first)
// 3. Limit graph traversal depth (2-3 hops max)
// 4. Batch queries where possible
```

---

## Monitoring & Quality

```typescript
// Metrics to track:
interface GraphRAGMetrics {
  // Performance
  query_latency_p95_ms: number;
  graph_retrieval_time_ms: number;
  semantic_retrieval_time_ms: number;
  llm_generation_time_ms: number;
  
  // Quality
  average_confidence_score: number;
  citation_count_per_query: number;
  user_satisfaction_rating: number;  // Thumbs up/down
  
  // Cost
  average_tokens_per_query: number;
  daily_query_count: number;
  estimated_daily_cost_usd: number;
  
  // Cache
  cache_hit_rate: number;
  conversation_continuation_rate: number;
}
```

---

## Security & Privacy

```typescript
// Row-level security: Filter graph results by tenant
const graphContext = await this.graphRetrieval.retrieveSubgraph(
  intent,
  { tenant_id: request.tenant_id }  // Only return data user can access
);

// PII scrubbing: Remove sensitive info from cached contexts
function scrubPII(context: string): string {
  return context
    .replace(/\b[\w\.-]+@[\w\.-]+\.\w{2,4}\b/g, '[EMAIL]')
    .replace(/\b\d{3}[-.]?\d{3}[-.]?\d{4}\b/g, '[PHONE]');
}

// Audit logging: Track all queries for compliance
await auditLog.record({
  user_id: request.user_id,
  query: request.query,
  data_accessed: graphContext.nodes.map(n => n.id),
  timestamp: new Date()
});
```

---

## Migration Path

### Phase 1: Basic GraphRAG (Months 1-2)
- Query understanding + intent extraction
- Graph retrieval (simple patterns)
- Answer generation with citations
- Redis caching

### Phase 2: Semantic Integration (Months 3-4)
- Pinecone semantic augmentation
- Hybrid graph + vector queries
- Confidence scoring
- Quality metrics

### Phase 3: Conversational (Months 5-6)
- Conversation context management
- Follow-up question handling
- Multi-turn reasoning
- User feedback loops

### Phase 4: Advanced (Months 7-9)
- Query optimization (cost + latency)
- Custom prompts per use case
- A/B testing different LLM models
- Real-time graph updates

---

## Alternatives Considered

### Alternative 1: LangChain with Graph Tools
**Pros:** Framework handles orchestration, many integrations
**Cons:** Generic agents slower, unpredictable costs, harder to optimize
**Decision:** Custom pipeline more control for our specific use case

### Alternative 2: Neo4j + GDS + Embedding Projections
**Pros:** Everything in Neo4j, single system
**Cons:** Neo4j vector search less mature than Pinecone, limited to Cypher
**Decision:** Pinecone better for semantic search

### Alternative 3: Pure LLM with Long Context
**Pros:** Simpler architecture, just LLM calls
**Cons:** Expensive ($0.10+ per query), slower, hallucinates, no structure
**Decision:** Hybrid approach better accuracy + cost

---

## Consequences

### Positive
- Natural language interface to complex graph queries
- Grounded answers (no hallucinations from LLM)
- Explainable (citations show reasoning)
- Combines structure (graph) + semantics (embeddings)
- Cost-effective (~$0.01 per query with caching)

### Negative
- Complex pipeline (5 steps, multiple services)
- Debugging harder (which component failed?)
- Query latency higher than pure DB queries (seconds vs milliseconds)
- LLM dependency (OpenAI/Anthropic availability)

### Neutral
- Requires tuning (prompt engineering, context assembly)
- Quality depends on underlying graph data quality
- Continuous monitoring needed

---

## Success Metrics

**Performance:**
- Query latency: p95 < 5 seconds
- Cache hit rate: >60%
- Concurrent queries: >100

**Quality:**
- User satisfaction: >4/5 stars
- Confidence score: average >0.7
- Citation accuracy: >95% (sources actually support claims)

**Cost:**
- Average cost per query: <$0.02
- Monthly cost at 10k queries: <$200

---

## References

- [Microsoft GraphRAG Paper](https://www.microsoft.com/en-us/research/blog/graphrag-unlocking-llm-discovery-on-narrative-private-data/)
- [Neo4j + LLM Integration Patterns](https://neo4j.com/labs/genai-ecosystem/)
- [RAG Best Practices](https://www.pinecone.io/learn/retrieval-augmented-generation/)
- [LangChain Graph QA](https://python.langchain.com/docs/use_cases/graph/)

---
