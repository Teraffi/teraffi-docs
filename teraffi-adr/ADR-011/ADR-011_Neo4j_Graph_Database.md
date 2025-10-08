# ADR-011: Neo4j Graph Database Architecture

**Status:** Proposed  
**Date:** 2025-10-01  
**Decision Makers:** Platform Architecture Team  
**Related ADRs:** ADR-001 (Runtime), ADR-012 (Vector Embeddings), ADR-013 (GraphRAG)

---

## Context

TERAFFI's Pillar 1 (Intelligence) requires modeling complex relationships between members, brands, content, cultural trends, and demographics. These relationships are:
- **Multi-hop**: Friend-of-friend networks, influence chains, recommendation paths
- **Weighted**: Affinity scores (0.0-1.0), alignment ratings, engagement metrics
- **Temporal**: Trends emerging/declining, partnership history, engagement over time
- **Bidirectional**: Mutual influence, reciprocal partnerships, collaborative relationships

Relational databases excel at structured data but struggle with:
- Variable-depth traversals (find all influences 3-5 hops away)
- Complex join patterns (multiple relationship types in single query)
- Graph algorithms (PageRank, community detection, shortest path)
- Schema flexibility (adding new relationship types without migrations)

Graph databases are purpose-built for these patterns.

### Requirements

**Functional:**
1. Store members, brands, content, trends, demographics, geographic entities
2. Model relationships: HAS_AFFINITY_WITH, PARTNERS_WITH, INFLUENCES, ALIGNS_WITH, APPEALS_TO, WORKS_FOR, CREATES, SPONSORS, REFLECTS
3. Support affinity queries: "Find brands aligned with this member's cultural interests"
4. Enable partnership discovery: "Recommend partners based on mutual affinities and successful patterns"
5. Track temporal patterns: "How has this brand's cultural alignment evolved?"
6. Integrate with PostgreSQL for transactional data

**Non-Functional:**
1. Sub-second query response for 2-3 hop traversals
2. Support 1M+ nodes, 10M+ relationships by Year 2
3. Handle 100+ concurrent affinity queries
4. 99.9% availability
5. Point-in-time backups daily

---

## Decision

**Use Neo4j 5.x Community/Enterprise as the primary graph database** with DozerDB extensions for enhanced query capabilities.

**Architecture:**
- Neo4j runs in dedicated Azure Container Instance (or VM for Enterprise)
- Standalone deployment initially; cluster for HA in Phase 2
- Redis on port 6380 for GraphRAG query cache
- Dual-write from platform services: PostgreSQL (transactional) + Neo4j (graph)
- Graph updates via events from outbox pattern (eventual consistency acceptable)

---

## Rationale

### Why Neo4j Over Alternatives

**Neo4j vs. PostgreSQL + pg_graph:**
- Native graph storage (not relational emulation)
- Cypher query language optimized for graph patterns
- Sub-linear traversal performance (constant time per hop vs. exponential joins)
- Rich graph algorithms library (PageRank, community detection, shortest path)

**Neo4j vs. Amazon Neptune:**
- Better Cypher support (Neptune's Gremlin-first, Cypher secondary)
- Simpler deployment (containerized, no AWS vendor lock-in)
- Stronger local development experience (Docker Compose)
- Better observability and tooling (Neo4j Browser, Bloom)

**Neo4j vs. ArangoDB:**
- More mature graph-specific features
- Larger community and ecosystem
- Better documentation for graph algorithms
- Proven at scale (LinkedIn, eBay, Cisco use Neo4j)

**Community vs. Enterprise:**
- Start with Community (free, sufficient for Phase 1)
- Migrate to Enterprise when clustering/RBAC needed (Phase 2+)
- Enterprise features: Multi-database, clustering, RBAC, LDAP/SSO

---

## Implementation Details

### Node Schema

```cypher
// Member nodes
CREATE CONSTRAINT member_id IF NOT EXISTS FOR (m:Member) REQUIRE m.id IS UNIQUE;
CREATE INDEX member_industry IF NOT EXISTS FOR (m:Member) ON (m.industry);
CREATE INDEX member_location IF NOT EXISTS FOR (m:Member) ON (m.location);

(:Member {
  id: 'mem_<uuid>',                    // UUID from PostgreSQL
  name: 'Sarah Bennett',
  email: 'sarah@example.com',          // For lookups
  company: 'Global Brands Inc',
  industry: 'consumer_goods',
  member_type: 'brand_executive',      // brand_executive | creator | retailer
  location: 'New York, NY',
  created_at: datetime(),
  updated_at: datetime(),
  pg_synced_at: datetime()             // Last sync from PostgreSQL
})

// Brand nodes
CREATE CONSTRAINT brand_id IF NOT EXISTS FOR (b:Brand) REQUIRE b.id IS UNIQUE;
CREATE INDEX brand_industry IF NOT EXISTS FOR (b:Brand) ON (b.industry);

(:Brand {
  id: 'brand_<uuid>',
  name: 'EcoWear',
  industry: 'sustainable_fashion',
  values: ['sustainability', 'authenticity', 'innovation'],
  annual_revenue: 500000000,
  employee_count: 1500,
  founded_year: 2015,
  website: 'https://ecowear.com',
  created_at: datetime(),
  updated_at: datetime()
})

// Content nodes
CREATE CONSTRAINT content_id IF NOT EXISTS FOR (c:Content) REQUIRE c.id IS UNIQUE;
CREATE INDEX content_platform IF NOT EXISTS FOR (c:Content) ON (c.platform);
CREATE INDEX content_genre IF NOT EXISTS FOR (c:Content) ON (c.genre);

(:Content {
  id: 'content_<uuid>',
  title: 'American Beauty Star',
  type: 'tv_series',                   // tv_series | podcast | youtube_channel | event
  platform: 'Lifetime',
  genre: ['reality', 'fashion'],
  audience_size: 2000000,
  engagement_rate: 0.087,
  avg_viewer_age: 34,
  premiere_date: date('2023-01-15'),
  status: 'active',                    // active | archived | upcoming
  created_at: datetime(),
  updated_at: datetime()
})

// Cultural Trend nodes
CREATE CONSTRAINT trend_id IF NOT EXISTS FOR (t:Trend) REQUIRE t.id IS UNIQUE;
CREATE INDEX trend_momentum IF NOT EXISTS FOR (t:Trend) ON (t.momentum_score);

(:Trend {
  id: 'trend_<uuid>',
  name: 'sustainable_fashion',
  category: 'lifestyle',               // lifestyle | technology | social | economic
  momentum_score: 0.89,                // 0.0-1.0, higher = gaining traction
  emergence_date: date('2023-01-01'),
  peak_date: date('2024-06-15'),       // Predicted or actual
  related_keywords: ['eco', 'circular economy', 'upcycling'],
  geographic_focus: ['North America', 'Europe'],
  demographic_appeal: ['millennials', 'gen_z'],
  created_at: datetime(),
  updated_at: datetime()
})

// Demographic nodes
CREATE CONSTRAINT demo_id IF NOT EXISTS FOR (d:Demographic) REQUIRE d.id IS UNIQUE;

(:Demographic {
  id: 'demo_<uuid>',
  segment: 'millennials',
  age_range: '25-40',
  characteristics: ['tech_savvy', 'value_conscious', 'experience_driven'],
  size: 72000000,                      // US population estimate
  purchasing_power: 'high',
  media_consumption: ['streaming', 'social', 'podcasts'],
  created_at: datetime(),
  updated_at: datetime()
})

// Geographic nodes
CREATE CONSTRAINT geo_id IF NOT EXISTS FOR (g:Geographic) REQUIRE g.id IS UNIQUE;

(:Geographic {
  id: 'geo_<uuid>',
  name: 'North America',
  type: 'region',                      // region | country | state | city
  market_size: 'large',
  gdp: 25000000000000,
  population: 580000000,
  created_at: datetime(),
  updated_at: datetime()
})
```

### Relationship Schema

```cypher
// Affinity relationships (Member ↔ Trend)
CREATE INDEX affinity_strength IF NOT EXISTS FOR ()-[r:HAS_AFFINITY_WITH]-() ON (r.strength);

(:Member)-[:HAS_AFFINITY_WITH {
  strength: 0.85,                      // 0.0-1.0, calculated by affinity engine
  confidence: 0.92,                    // How confident we are in this score
  basis: ['stated_goals', 'past_partnerships', 'engagement_signals'],
  calculated_at: datetime(),
  expires_at: datetime() + duration({days: 30}),  // Recalculate monthly
  signals: {
    explicit_interest: true,
    engagement_count: 47,
    recent_activity: datetime()
  }
}]->(:Trend)

// Brand alignment (Brand ↔ Trend)
CREATE INDEX alignment_score IF NOT EXISTS FOR ()-[r:ALIGNS_WITH]-() ON (r.alignment_score);

(:Brand)-[:ALIGNS_WITH {
  alignment_score: 0.92,               // 0.0-1.0
  authenticity_score: 0.88,            // Genuine vs. opportunistic
  evidence: ['brand_values', 'product_line', 'marketing_campaigns'],
  validated_by: 'cultural_intelligence_ai',
  validated_at: datetime(),
  market_perception: 'strong',         // strong | moderate | weak | negative
  risk_flags: []                       // Empty = safe, otherwise concerns
}]->(:Trend)

// Content appeal (Content ↔ Demographic)
CREATE INDEX engagement_rate IF NOT EXISTS FOR ()-[r:APPEALS_TO]-() ON (r.engagement_rate);

(:Content)-[:APPEALS_TO {
  engagement_rate: 0.087,              // Click-through, watch-time, etc.
  audience_overlap: 0.73,              // % of content audience in this demographic
  sentiment: 'positive',               // positive | neutral | negative
  top_themes: ['empowerment', 'creativity', 'competition'],
  measured_at: datetime(),
  sample_size: 250000
}]->(:Demographic)

// Employment (Member ↔ Brand)
(:Member)-[:WORKS_FOR {
  title: 'VP of Marketing',
  start_date: date('2020-01-15'),
  current: true,
  verified: true,
  source: 'linkedin'
}]->(:Brand)

// Content creation (Member ↔ Content)
(:Member)-[:CREATES {
  role: 'executive_producer',          // creator | producer | host | contributor
  start_date: date('2023-01-01'),
  end_date: date('2024-12-31'),
  ownership_stake: 0.15,               // Equity or profit share
  verified: true
}]->(:Content)

// Sponsorship (Brand ↔ Content)
CREATE INDEX sponsorship_amount IF NOT EXISTS FOR ()-[r:SPONSORS]-() ON (r.amount);

(:Brand)-[:SPONSORS {
  amount: 250000,
  currency: 'USD',
  duration_months: 6,
  start_date: date('2024-01-01'),
  end_date: date('2024-06-30'),
  deliverables: ['product_placement', 'branded_segment', 'social_mentions'],
  performance_metrics: {
    impressions: 15000000,
    engagement_rate: 0.042,
    brand_lift: 0.18
  },
  roi_score: 3.2,                      // Revenue multiple
  status: 'active'                     // active | completed | cancelled
}]->(:Content)

// Partnership (Member ↔ Member)
CREATE INDEX partnership_status IF NOT EXISTS FOR ()-[r:PARTNERS_WITH]-() ON (r.status);
CREATE INDEX partnership_value IF NOT EXISTS FOR ()-[r:PARTNERS_WITH]-() ON (r.total_value);

(:Member)-[:PARTNERS_WITH {
  partnership_id: 'part_<uuid>',       // Links to PostgreSQL partnerships table
  status: 'active',                    // pending | active | completed | cancelled
  success_score: 4.3,                  // 0-5 based on milestones/outcomes
  total_value: 250000,
  start_date: date('2024-01-15'),
  end_date: date('2024-12-31'),
  type: 'brand_collaboration',         // brand_collaboration | co_marketing | sponsored_content
  mutual_benefit: 0.87,                // Fairness score
  created_at: datetime(),
  updated_at: datetime()
}]->(:Member)

// Cultural influence (Trend ↔ Demographic)
(:Trend)-[:INFLUENCES {
  influence_strength: 0.79,            // How strongly trend affects this demographic
  adoption_rate: 0.45,                 // % of demographic engaged with trend
  velocity: 'accelerating',            // accelerating | stable | declining
  geographic_concentration: ['urban', 'coastal'],
  measured_at: datetime()
}]->(:Demographic)

// Cultural reflection (Content ↔ Trend)
(:Content)-[:REFLECTS {
  reflection_strength: 0.91,           // How well content embodies trend
  authenticity: 0.88,                  // Genuine vs. forced
  pioneered: false,                    // Did content create trend or follow?
  validation_source: 'ai_analysis',
  validated_at: datetime()
}]->(:Trend)
```

### Query Patterns

**1. Affinity-Based Partnership Discovery**
```cypher
// Find potential brand partners for a member based on shared cultural affinities
MATCH (m:Member {id: $memberId})-[aff:HAS_AFFINITY_WITH]->(t:Trend)<-[align:ALIGNS_WITH]-(b:Brand)
WHERE aff.strength > 0.7 AND align.alignment_score > 0.75
WITH b, t, aff.strength * align.alignment_score AS compatibility_score
MATCH (b)<-[:WORKS_FOR]-(contact:Member)
WHERE contact.member_type = 'brand_executive'
RETURN b.name AS brand,
       b.industry AS industry,
       collect(DISTINCT t.name) AS shared_trends,
       avg(compatibility_score) AS avg_compatibility,
       collect(DISTINCT {name: contact.name, email: contact.email}) AS contacts
ORDER BY avg_compatibility DESC
LIMIT 10;
```

**2. Partnership Success Prediction**
```cypher
// Predict success of potential partnership based on historical patterns
MATCH path = (m1:Member)-[:PARTNERS_WITH*1..2]-(m2:Member)-[:PARTNERS_WITH {status: 'completed'}]-(m3:Member)
WHERE m1.id = $member1Id AND m3.id = $member2Id
WITH m2, path, length(path) AS distance
MATCH (m2)-[p:PARTNERS_WITH {status: 'completed'}]-()
RETURN avg(p.success_score) AS predicted_success_score,
       count(p) AS historical_partnerships,
       min(distance) AS connection_distance,
       collect(DISTINCT m2.name) AS mutual_connections
ORDER BY predicted_success_score DESC;
```

**3. Cultural Trend Discovery**
```cypher
// Find emerging trends relevant to a brand's industry and values
MATCH (b:Brand {id: $brandId})
MATCH (t:Trend)
WHERE t.momentum_score > 0.8
  AND t.emergence_date > date() - duration({months: 6})
  AND any(value IN b.values WHERE value IN t.related_keywords)
MATCH (t)-[:INFLUENCES]->(d:Demographic)
MATCH (b)-[:SPONSORS]->(c:Content)-[:APPEALS_TO]->(d)
WITH t, b, count(DISTINCT d) AS demographic_overlap, avg(c.engagement_rate) AS avg_engagement
RETURN t.name AS trend,
       t.momentum_score AS momentum,
       t.category AS category,
       demographic_overlap,
       avg_engagement,
       (t.momentum_score * demographic_overlap * avg_engagement) AS relevance_score
ORDER BY relevance_score DESC
LIMIT 5;
```

**4. Multi-Hop Influence Analysis**
```cypher
// Find influential nodes within 3 hops of a member (PageRank-style)
CALL gds.pageRank.stream({
  nodeProjection: ['Member', 'Brand', 'Content'],
  relationshipProjection: {
    INFLUENCES: {type: 'INFLUENCES', orientation: 'NATURAL'},
    PARTNERS_WITH: {type: 'PARTNERS_WITH', orientation: 'UNDIRECTED'}
  },
  maxIterations: 20,
  dampingFactor: 0.85
})
YIELD nodeId, score
WITH gds.util.asNode(nodeId) AS node, score
WHERE distance(node, (m:Member {id: $memberId})) <= 3
RETURN labels(node)[0] AS type,
       node.name AS name,
       score AS influence_score
ORDER BY score DESC
LIMIT 20;
```

**5. Community Detection for Market Segmentation**
```cypher
// Identify clusters of brands/members with similar affinities (Louvain algorithm)
CALL gds.louvain.stream({
  nodeProjection: ['Member', 'Brand'],
  relationshipProjection: {
    HAS_AFFINITY_WITH: {type: 'HAS_AFFINITY_WITH'},
    ALIGNS_WITH: {type: 'ALIGNS_WITH'}
  }
})
YIELD nodeId, communityId
WITH communityId, collect(gds.util.asNode(nodeId)) AS members
WHERE size(members) > 5
RETURN communityId,
       size(members) AS community_size,
       [m IN members | {name: m.name, type: labels(m)[0]}] AS community_members
ORDER BY community_size DESC;
```

---

## Data Synchronization Strategy

### PostgreSQL → Neo4j (Dual Write)

**Pattern: Event-Driven Sync via Outbox**

```typescript
// When member profile updated in PostgreSQL
async function updateMemberProfile(memberId: string, updates: Partial<MemberProfile>) {
  await db.transaction(async (trx) => {
    // 1. Update PostgreSQL
    await trx.updateTable('member_profiles')
      .set(updates)
      .where('member_id', '=', memberId)
      .execute();
    
    // 2. Insert outbox event
    await trx.insertInto('outbox')
      .values({
        tenant_id: updates.tenant_id,
        topic: 'member.profile_updated',
        event_id: generateId(),
        payload: { memberId, updates },
        created_at: new Date(),
      })
      .execute();
  });
}

// Outbox relay publishes to Service Bus
// Neo4j sync worker consumes events
async function handleMemberProfileUpdated(event: MemberProfileUpdatedEvent) {
  const session = neo4jDriver.session();
  try {
    await session.run(`
      MERGE (m:Member {id: $memberId})
      SET m += $updates,
          m.pg_synced_at = datetime()
    `, {
      memberId: event.memberId,
      updates: {
        name: event.updates.name,
        company: event.updates.company,
        industry: event.updates.industry,
        location: event.updates.location,
        updated_at: new Date().toISOString(),
      }
    });
  } finally {
    await session.close();
  }
}
```

**Sync Events:**
- `member.created` → Create `:Member` node
- `member.profile_updated` → Update `:Member` properties
- `member.deleted` → Delete `:Member` node (cascade relationships)
- `partnership.created` → Create `:PARTNERS_WITH` relationship
- `partnership.stage_changed` → Update relationship properties
- `partnership.completed` → Update `success_score` based on outcomes

### Neo4j → PostgreSQL (Read-Only Reporting)

For analytics, periodically export graph metrics to PostgreSQL:
```cypher
// Weekly batch job: Export affinity scores to reporting table
MATCH (m:Member)-[aff:HAS_AFFINITY_WITH]->(t:Trend)
RETURN m.id AS member_id,
       t.id AS trend_id,
       aff.strength AS affinity_strength,
       aff.calculated_at AS calculated_at;
```

Insert into `affinity_scores` table for BI tools (Metabase, Looker).

---

## Performance Optimization

### Indexing Strategy

```cypher
// Constraint indexes (automatic B-tree)
CREATE CONSTRAINT member_id IF NOT EXISTS FOR (m:Member) REQUIRE m.id IS UNIQUE;
CREATE CONSTRAINT brand_id IF NOT EXISTS FOR (b:Brand) REQUIRE b.id IS UNIQUE;
CREATE CONSTRAINT content_id IF NOT EXISTS FOR (c:Content) REQUIRE c.id IS UNIQUE;
CREATE CONSTRAINT trend_id IF NOT EXISTS FOR (t:Trend) REQUIRE t.id IS UNIQUE;

// Property indexes for common filters
CREATE INDEX member_industry IF NOT EXISTS FOR (m:Member) ON (m.industry);
CREATE INDEX member_location IF NOT EXISTS FOR (m:Member) ON (m.location);
CREATE INDEX brand_industry IF NOT EXISTS FOR (b:Brand) ON (b.industry);
CREATE INDEX trend_momentum IF NOT EXISTS FOR (t:Trend) ON (t.momentum_score);
CREATE INDEX trend_category IF NOT EXISTS FOR (t:Trend) ON (t.category);

// Relationship indexes for scoring/filtering
CREATE INDEX affinity_strength IF NOT EXISTS FOR ()-[r:HAS_AFFINITY_WITH]-() ON (r.strength);
CREATE INDEX alignment_score IF NOT EXISTS FOR ()-[r:ALIGNS_WITH]-() ON (r.alignment_score);
CREATE INDEX partnership_status IF NOT EXISTS FOR ()-[r:PARTNERS_WITH]-() ON (r.status);
CREATE INDEX sponsorship_amount IF NOT EXISTS FOR ()-[r:SPONSORS]-() ON (r.amount);

// Composite indexes for common query patterns (Neo4j 5.x)
CREATE INDEX member_industry_location IF NOT EXISTS FOR (m:Member) ON (m.industry, m.location);
```

### Query Optimization Techniques

**1. Use Parameters (Prepared Statements)**
```cypher
// ✅ Good: Parameterized
MATCH (m:Member {id: $memberId})-[aff:HAS_AFFINITY_WITH]->(t:Trend)
WHERE aff.strength > $minStrength
RETURN t;

// ❌ Bad: String interpolation (no query plan cache)
MATCH (m:Member {id: 'mem_123'})-[aff:HAS_AFFINITY_WITH]->(t:Trend)
WHERE aff.strength > 0.7
RETURN t;
```

**2. Limit Traversal Depth**
```cypher
// ✅ Good: Bounded depth
MATCH path = (m:Member {id: $memberId})-[*1..3]-(other:Member)
RETURN other LIMIT 20;

// ❌ Bad: Unbounded (can explode)
MATCH path = (m:Member {id: $memberId})-[*]-(other:Member)
RETURN other;
```

**3. Use PROFILE for Query Analysis**
```cypher
PROFILE
MATCH (m:Member)-[aff:HAS_AFFINITY_WITH]->(t:Trend)
WHERE aff.strength > 0.8
RETURN m, t;
// Check db hits, rows processed in query plan
```

**4. Denormalize Hot Properties**
```cypher
// Store frequently accessed properties on nodes vs. fetching from PostgreSQL
(:Member {
  id: 'mem_123',
  name: 'Sarah Bennett',
  industry: 'consumer_goods',  // Denormalized for fast filtering
  // Full profile in PostgreSQL
})
```

### Caching Strategy

**Query Result Cache (Redis on port 6380)**
```typescript
// Cache expensive graph queries for GraphRAG
async function getAffinityRecommendations(memberId: string): Promise<AffinityMatch[]> {
  const cacheKey = `affinity:${memberId}`;
  
  // Check cache
  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);
  
  // Query Neo4j
  const session = neo4jDriver.session();
  const result = await session.run(affinityQuery, { memberId });
  const matches = result.records.map(parseAffinityRecord);
  await session.close();
  
  // Cache for 1 hour (affinities recalculated daily)
  await redis.setex(cacheKey, 3600, JSON.stringify(matches));
  
  return matches;
}
```

**Materialized Aggregates**
```cypher
// Pre-compute expensive aggregations nightly
MATCH (m:Member)-[:PARTNERS_WITH {status: 'completed'}]-(partner:Member)
WITH m, avg(partner.success_score) AS avg_partner_success
SET m.avg_partner_success_score = avg_partner_success,
    m.agg_computed_at = datetime();
```

---

## Operational Considerations

### Backup & Recovery

**Daily Backups:**
```bash
# Automated backup script (cron daily at 2 AM)
neo4j-admin database dump neo4j \
  --to-path=/backups/neo4j-$(date +%Y%m%d).dump

# Retention: 7 daily, 4 weekly, 12 monthly
```

**Restore Procedure:**
```bash
# Stop Neo4j
systemctl stop neo4j

# Restore from dump
neo4j-admin database load neo4j \
  --from-path=/backups/neo4j-20241001.dump \
  --overwrite-destination=true

# Restart
systemctl start neo4j
```

**Disaster Recovery:**
- Backups replicated to Azure Blob Storage (geo-redundant)
- RTO: 4 hours (restore + rebuild indexes)
- RPO: 24 hours (daily backups)
- For critical use cases: Enable Neo4j clustering (Enterprise) with sync replication

### Monitoring & Alerts

**Metrics to Track:**
```cypher
// Query performance (run via monitoring agent)
CALL dbms.queryJmx('org.neo4j:instance=kernel#0,name=Transactions')
YIELD attributes
RETURN attributes.PeakNumberOfConcurrentTransactions AS peak_txns,
       attributes.NumberOfOpenTransactions AS open_txns;

CALL dbms.queryJmx('org.neo4j:instance=kernel#0,name=Page cache')
YIELD attributes
RETURN attributes.BytesRead AS bytes_read,
       attributes.BytesWritten AS bytes_written,
       attributes.Faults AS page_faults;
```

**Prometheus Metrics (Neo4j JMX Exporter):**
- `neo4j_database_page_cache_hit_ratio` (target >95%)
- `neo4j_database_transaction_active_count` (alert if >100)
- `neo4j_database_store_size_bytes` (track growth)
- `neo4j_bolt_connections_opened_total` (connection pool health)

**Alerting Rules:**
```yaml
# Prometheus alert rules
groups:
  - name: neo4j_alerts
    rules:
      - alert: Neo4jCacheHitRatioLow
        expr: neo4j_database_page_cache_hit_ratio < 0.90
        for: 5m
        annotations:
          summary: "Neo4j cache hit ratio below 90%"
          
      - alert: Neo4jHighTransactionCount
        expr: neo4j_database_transaction_active_count > 100
        for: 2m
        annotations:
          summary: "Neo4j has >100 active transactions"
          
      - alert: Neo4jConnectionPoolExhausted
        expr: rate(neo4j_bolt_connections_opened_total[5m]) > 50
        annotations:
          summary: "Neo4j connection pool under pressure"
```

### Security

**Authentication:**
```bash
# Create application user with limited privileges
CREATE USER neo4j_app_user SET PASSWORD 'strong_password' CHANGE NOT REQUIRED;
GRANT ROLE reader TO neo4j_app_user;
GRANT ROLE writer TO neo4j_app_user;
DENY CREATE NEW LABEL ON GRAPH * TO neo4j_app_user;
DENY CREATE NEW NODE LABEL ON GRAPH * TO neo4j_app_user;
```

**Network Security:**
- Neo4j exposed only to backend services (not public internet)
- Bolt protocol (port 7687) over TLS
- HTTP API disabled in production
- Connection from Azure Container Apps via private endpoint (Phase 2)

**Audit Logging (Enterprise):**
```conf
# neo4j.conf
dbms.security.procedures.unrestricted=apoc.*
dbms.logs.security.level=INFO
dbms.security.auth_enabled=true
```

---

## Migration Path

### Phase 1: Foundation (Months 1-3)
- Deploy Neo4j Community in Azure Container Instance
- Implement core node types (Member, Brand, Trend)
- Build sync worker for `member.created` events
- Create affinity query endpoints (basic)

### Phase 2: Expansion (Months 4-6)
- Add Content, Demographic, Geographic nodes
- Implement all relationship types
- Build partnership discovery queries
- Add Redis query cache (port 6380)

### Phase 3: Advanced (Months 7-9)
- Graph algorithms (PageRank, Louvain)
- Temporal trend analysis
- Predictive scoring models
- Performance optimization (indexes, caching)

### Phase 4: Enterprise (Months 10-12)
- Migrate to Neo4j Enterprise
- Enable clustering (3-node causal cluster)
- Advanced monitoring & alerting
- Multi-region replication

---

## Alternatives Considered

### Alternative 1: PostgreSQL with Recursive CTEs
**Pros:** No additional database, familiar SQL
**Cons:** Poor performance for multi-hop queries (>3 hops), complex recursive CTEs, no graph algorithms

### Alternative 2: Amazon Neptune
**Pros:** Managed service, auto-scaling
**Cons:** AWS vendor lock-in, Gremlin-first (Cypher support weaker), higher cost, complex local dev

### Alternative 3: ArangoDB
**Pros:** Multi-model (graph + document), good performance
**Cons:** Smaller community, less mature graph algorithms, AQL vs Cypher learning curve

**Decision:** Neo4j offers best balance of maturity, community, graph-specific features, and operational simplicity.

---

## Consequences

### Positive
- Purpose-built for relationship queries (sub-second multi-hop traversals)
- Rich graph algorithms (PageRank, community detection, shortest path)
- Cypher query language is intuitive and powerful
- Strong community and ecosystem (APOC library, Neo4j Bloom)
- Proven at scale (LinkedIn, Airbnb, eBay use Neo4j)

### Negative
- Additional database to operate and monitor
- Eventual consistency between PostgreSQL and Neo4j (acceptable for affinity use cases)
- Learning curve for team (Cypher vs SQL)
- Cost of Neo4j Enterprise for clustering (deferred to Phase 4)

### Neutral
- Dual-write complexity managed via outbox pattern (already in architecture)
- Backup/restore procedures straightforward
- Can start with Community edition, upgrade to Enterprise when needed

---

## Success Metrics

**Performance:**
- 2-hop affinity queries: p95 < 100ms
- 3-hop influence queries: p95 < 500ms
- Partnership discovery (complex): p95 < 2s
- Cache hit ratio: >95%

**Reliability:**
- Uptime: 99.9%
- Data sync lag: <5 minutes (p95)
- Zero data loss from sync failures

**Scale:**
- Support 1M+ nodes by Month 12
- Support 10M+ relationships by Month 12
- Handle 100+ concurrent queries

---

## References

- [Neo4j Cypher Manual](https://neo4j.com/docs/cypher-manual/current/)
- [Neo4j Operations Manual](https://neo4j.com/docs/operations-manual/current/)
- [Graph Algorithms Library](https://neo4j.com/docs/graph-data-science/current/)
- [DozerDB Documentation](https://dozerdb.org/)
- [Neo4j in Production: Best Practices](https://neo4j.com/developer/in-production/)

---
