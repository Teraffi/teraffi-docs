# ADR-015: Affinity Scoring Algorithm

**Status:** Proposed  
**Date:** 2025-10-01  
**Decision Makers:** Platform Architecture Team  
**Related ADRs:** ADR-011 (Neo4j), ADR-012 (Pinecone), ADR-014 (LLM Gateway)

---

## Context

TERAFFI's **Affinity Acceleration Framework™** requires an intelligent scoring system that powers the four stages: Discovery, Matching, Activation, and Amplification. The system must answer two fundamental questions:

1. **"Who are you?"** - Understanding brand/IP DNA through affinities
2. **"What do you want?"** - Aligning goals with partnership opportunities

**The Affinity Engine™** uses semantic triples and graph relationships to identify high-value partnerships that traditional methods miss.

### Core Requirements from Framework

**Affinity Discovery™:**
- Map core affinities: cultural relevance, values, audience psychographics, emotional connections
- Identity calibration: capture brand mission, voice, and unique factors that drive engagement
- Go beyond surface metrics to understand what makes brands resonate

**Affinity Matching™:**
- Proprietary algorithm for synergy analysis
- Find complementary strengths that amplify value
- Evaluate shared values, audiences, and goals for authentic partnerships

**Time-Stamped Goal Tracking:**
- First-party data integration with timestamps
- Goals as primary drivers: target market expansion, revenue growth, brand elevation, IP monetization
- Track goal evolution over time

**Cultural Trend Integration:**
- Real-time cultural shift monitoring
- Predictive partnership creation ("skate to where the puck is going")
- Recognition of consumer behavior changes

**Deal Data Feedback Loop:**
- Continuous learning from partnership outcomes
- Performance metrics: engagement, revenue growth, audience response
- Dynamic recalibration based on market shifts

### Requirements

**Functional:**
1. Extract semantic triples from natural language input
2. Calculate affinity scores between entities (0-100 scale)
3. Combine multiple signals: semantic similarity, goal alignment, cultural momentum, historical performance
4. Explain scores (which factors contributed most)
5. Update scores as new data arrives
6. Support real-time queries (<2s for cached, <30s for new)

**Non-Functional:**
1. Processing time: 20-30s first-time, 1-2s subsequent
2. Accuracy: >75% correlation with successful partnerships
3. Freshness: scores updated within 24 hours of new data
4. Explainability: top 3 factors contributing to score
5. Scalability: compute 1M+ affinity scores

---

## Decision

**Implement the Affinity Engine™** as a hybrid scoring system combining:

1. **Semantic Triple Analysis** (graph-based knowledge)
2. **Goal Alignment Scoring** (time-stamped first-party data)
3. **Cultural Trend Momentum** (real-time market intelligence)
4. **Historical Deal Performance** (feedback loop learning)
5. **LLM-Enhanced Authenticity** (genuine vs opportunistic detection)

**Core Architecture:**
```
Natural Language Input ("Who are you?" + "What do you want?")
    ↓
Triple Extraction (semantic relationships)
    ↓
Knowledge Graph Population (Neo4j)
    ↓
Affinity Score Calculation
    ├─ Semantic Similarity (35%)
    ├─ Goal Alignment (30%)
    ├─ Cultural Momentum (20%)
    ├─ Historical Success (10%)
    └─ Authenticity Check (5%)
    ↓
Ranked Partnership Recommendations
```

**Technology Stack:**
- **Neo4j** (ADR-011): Graph database for semantic triples
- **Pinecone** (ADR-012): Vector embeddings for semantic similarity
- **LLM Gateway** (ADR-014): Triple extraction and authenticity analysis
- **Redis**: Score caching (1-2 second subsequent queries)
- **PostgreSQL**: Deal outcomes and feedback loop

---

## Rationale

### Alignment with Framework Documents

**From Affinity Acceleration Framework™:**
- ✅ Four-stage process (Discovery → Matching → Activation → Amplification)
- ✅ AI-driven but human-enhanced intelligence
- ✅ Continuous feedback loops for exponential growth
- ✅ Focus on authentic, resonant partnerships

**From Technical Specifications:**
- ✅ Time-stamped first-party data as primary driver
- ✅ Real-time cultural trend monitoring
- ✅ Predictive insights for future affinities
- ✅ Deal data feedback for model improvement
- ✅ Scalable decision-making framework

**From Affinity Engine Proposal:**
- ✅ Two-question simplicity ("Who are you?" + "What do you want?")
- ✅ Semantic triple extraction and knowledge graph
- ✅ 20-30 second initial processing, 1-2 second subsequent searches
- ✅ Self-improving system with emergent knowledge properties
- ✅ Zero cold start via public data enrichment

### Why This Approach Wins

**Deep Affinity Science:**
- Analyzes emotional and cultural affinities beyond surface metrics
- Creates sustainable, resonant growth through authentic connections

**Tech-Driven Scalability:**
- Proprietary algorithm scales across industries
- Data network effects: more deals → better matches → more users

**Predictive vs Reactive:**
- Identifies future opportunities before they're obvious
- Cultural intelligence anticipates market movements

**Zero Cold Start:**
- Public data ingestion (Wikipedia, IMDb, social media, earnings reports)
- Immediate value from day one, improves continuously

**Systems Theory & Emergent Knowledge:**
- Individual triples represent atomic facts
- At scale, interconnected triples create emergent knowledge structures
- More triples → more detailed systemic understanding
- Knowledge substrate enables inference, pattern identification, prediction

---

## Implementation Details

### 1. Natural Language Processing & Triple Extraction

```typescript
// packages/affinity-engine/src/triple-extraction.ts

interface UserInput {
  question1: string;  // "Who are you?"
  question2: string;  // "What do you want?"
  user_id: string;
  timestamp: Date;
}

interface ExtractedTriples {
  facts: Triple[];
  people: Triple[];
  places: Triple[];
  things: Triple[];
  brands: Triple[];
  emotions: Triple[];
  colors: Triple[];
  genres: Triple[];
  demographics: Triple[];
}

interface Triple {
  subject: string;
  predicate: string;
  object: string;
  confidence: number;
  source: string;  // 'user_input' | 'wikipedia' | 'imdb' | 'social_media'
  timestamp: Date;
}

export class TripleExtractor {
  constructor(
    private llmGateway: LLMGatewayClient,
    private publicDataSources: PublicDataAggregator
  ) {}

  async extractFromInput(input: UserInput): Promise<ExtractedTriples> {
    // 1. Parse natural language using LLM
    const structuredData = await this.llmGateway.complete({
      model: 'gpt-4o',
      messages: [{
        role: 'system',
        content: `Extract semantic triples from user input. Format as JSON with categories:
facts, people, places, things, brands, emotions, colors, genres, demographics.

Each triple: [subject, predicate, object]

Examples:
"I produce American Beauty Star" →
{
  "facts": [
    ["user", "produces", "American Beauty Star"],
    ["American Beauty Star", "is_type", "TV show"]
  ],
  "genres": [
    ["American Beauty Star", "belongs_to", "reality competition"],
    ["American Beauty Star", "focuses_on", "beauty industry"]
  ],
  "demographics": [
    ["American Beauty Star", "appeals_to", "women aged 18-34"],
    ["American Beauty Star", "targets", "beauty enthusiasts"]
  ]
}`
      }, {
        role: 'user',
        content: `Question 1 (Who are you?): ${input.question1}
Question 2 (What do you want?): ${input.question2}`
      }],
      response_format: { type: 'json_object' },
      temperature: 0.1
    });

    const userTriples = JSON.parse(structuredData.choices[0].message.content);

    // 2. Enrich with public data sources (run in parallel)
    const publicTriples = await this.enrichWithPublicData(userTriples);

    // 3. Combine and deduplicate
    return this.mergeTriples(userTriples, publicTriples);
  }

  private async enrichWithPublicData(
    userTriples: ExtractedTriples
  ): Promise<ExtractedTriples> {
    // Extract entities mentioned by user
    const entities = this.extractEntityNames(userTriples);

    // Query public sources in parallel
    const [wikipedia, imdb, social, earnings] = await Promise.all([
      this.publicDataSources.queryWikipedia(entities),
      this.publicDataSources.queryIMDB(entities),
      this.publicDataSources.querySocialMedia(entities),
      this.publicDataSources.queryEarningsReports(entities)
    ]);

    return this.aggregatePublicTriples(wikipedia, imdb, social, earnings);
  }

  private extractEntityNames(triples: ExtractedTriples): string[] {
    const entities = new Set<string>();
    
    for (const category of Object.values(triples)) {
      for (const triple of category) {
        entities.add(triple.subject);
        entities.add(triple.object);
      }
    }
    
    return Array.from(entities);
  }
}
```

### 2. Knowledge Graph Population

```typescript
// packages/affinity-engine/src/graph-population.ts

export class KnowledgeGraphPopulator {
  constructor(private neo4j: Neo4jClient) {}

  async populateGraph(triples: ExtractedTriples, userId: string): Promise<void> {
    const session = this.neo4j.session();
    
    try {
      // Create nodes and relationships from triples
      for (const category of Object.keys(triples)) {
        for (const triple of triples[category]) {
          await session.run(`
            MERGE (s {id: $subject})
            SET s.name = $subject,
                s.type = $category,
                s.last_updated = datetime()
            
            MERGE (o {id: $object})
            SET o.name = $object,
                o.last_updated = datetime()
            
            MERGE (s)-[r:${this.normalizeRelation(triple.predicate)}]->(o)
            SET r.confidence = $confidence,
                r.source = $source,
                r.created_at = datetime($timestamp),
                r.user_id = $userId
          `, {
            subject: triple.subject,
            object: triple.object,
            category,
            confidence: triple.confidence,
            source: triple.source,
            timestamp: triple.timestamp.toISOString(),
            userId
          });
        }
      }

      // Create embeddings for entities (for semantic search)
      await this.createEntityEmbeddings(triples, userId);
    } finally {
      await session.close();
    }
  }

  private normalizeRelation(predicate: string): string {
    // Convert natural language predicates to graph relations
    const relationMap: Record<string, string> = {
      'produces': 'PRODUCES',
      'is_type': 'IS_TYPE',
      'belongs_to': 'BELONGS_TO',
      'targets': 'TARGETS',
      'appeals_to': 'APPEALS_TO',
      'hosted_by': 'HOSTED_BY',
      'sponsored': 'SPONSORED',
      'aligns_with': 'ALIGNS_WITH',
      'associated_with': 'ASSOCIATED_WITH',
      'appears_in': 'APPEARS_IN',
      'targets_audience': 'TARGETS_AUDIENCE'
    };
    
    return relationMap[predicate] || 'RELATED_TO';
  }

  private async createEntityEmbeddings(
    triples: ExtractedTriples,
    userId: string
  ): Promise<void> {
    // Extract unique entities
    const entities = new Set<string>();
    for (const category of Object.values(triples)) {
      for (const triple of category) {
        entities.add(triple.subject);
        entities.add(triple.object);
      }
    }

    // Create text representations for embedding
    const entityTexts = Array.from(entities).map(entity => {
      const relevantTriples = this.findTriplesForEntity(entity, triples);
      return this.formatEntityForEmbedding(entity, relevantTriples);
    });

    // Generate embeddings and store in Pinecone (handled by embedding service)
    // This will be called asynchronously via event queue
  }
}
```

### 3. Goal Alignment Scoring

```typescript
// packages/affinity-engine/src/components/goal-alignment.ts

interface Goal {
  type: 'market_expansion' | 'revenue_growth' | 'brand_elevation' | 'ip_monetization';
  description: string;
  target_value?: number;
  target_demographics?: string[];
  target_geography?: string[];
  timeline?: Date;
  timestamp: Date;  // When goal was set/updated
}

export class GoalAlignmentScorer {
  constructor(
    private neo4j: Neo4jClient,
    private pinecone: PineconeClient,
    private llmGateway: LLMGatewayClient
  ) {}

  async calculateAlignment(
    userGoals: Goal[],
    partnerEntity: string
  ): Promise<number> {
    // Weight recent goals higher (exponential decay)
    const weightedGoals = this.applyTemporalWeighting(userGoals);

    const alignmentScores = await Promise.all(
      weightedGoals.map(async (goal) => {
        // Check if partner can help achieve this goal
        const canFulfill = await this.evaluateGoalFulfillment(goal, partnerEntity);
        return canFulfill.score * goal.weight;
      })
    );

    return alignmentScores.reduce((sum, score) => sum + score, 0) / alignmentScores.length;
  }

  private applyTemporalWeighting(goals: Goal[]): Array<Goal & { weight: number }> {
    const now = new Date();
    
    return goals.map(goal => {
      const ageInDays = differenceInDays(now, goal.timestamp);
      // Exponential decay: weight = e^(-age/90)
      // Goals older than 90 days have ~37% weight
      const weight = Math.exp(-ageInDays / 90);
      
      return { ...goal, weight };
    });
  }

  private async evaluateGoalFulfillment(
    goal: Goal,
    partnerEntity: string
  ): Promise<{ score: number; reasoning: string }> {
    const session = this.neo4j.session();
    
    try {
      // Query graph for partner's capabilities
      const result = await session.run(`
        MATCH (partner {id: $partnerId})
        OPTIONAL MATCH (partner)-[:TARGETS]->(demo:Demographic)
        OPTIONAL MATCH (partner)-[:OPERATES_IN]->(geo:Geographic)
        OPTIONAL MATCH (partner)-[:HAS_CAPABILITY]->(cap)
        
        RETURN 
          collect(DISTINCT demo.segment) as demographics,
          collect(DISTINCT geo.name) as geographies,
          collect(DISTINCT cap.name) as capabilities,
          partner.name as partner_name,
          partner.industry as industry
      `, { partnerId: partnerEntity });

      const record = result.records[0];
      const partnerDemographics = record.get('demographics');
      const partnerGeographies = record.get('geographies');
      const partnerCapabilities = record.get('capabilities');
      const partnerName = record.get('partner_name');
      const industry = record.get('industry');

      // Calculate overlap
      let score = 0;
      let reasoning = [];

      if (goal.target_demographics && partnerDemographics.length > 0) {
        const demoOverlap = this.calculateOverlap(
          goal.target_demographics,
          partnerDemographics
        );
        score += demoOverlap * 0.4;
        reasoning.push(`${(demoOverlap * 100).toFixed(0)}% demographic overlap`);
      }

      if (goal.target_geography && partnerGeographies.length > 0) {
        const geoOverlap = this.calculateOverlap(
          goal.target_geography,
          partnerGeographies
        );
        score += geoOverlap * 0.3;
        reasoning.push(`${(geoOverlap * 100).toFixed(0)}% geographic overlap`);
      }

      // LLM evaluation for qualitative alignment
      const qualitativeScore = await this.llmEvaluateGoalFit(
        goal,
        { name: partnerName, industry, capabilities: partnerCapabilities }
      );
      score += qualitativeScore * 0.3;
      reasoning.push(`Qualitative fit: ${(qualitativeScore * 100).toFixed(0)}%`);

      return {
        score: Math.max(0, Math.min(1, score)),
        reasoning: reasoning.join('; ')
      };
    } finally {
      await session.close();
    }
  }

  private async llmEvaluateGoalFit(
    goal: Goal,
    partner: { name: string; industry: string; capabilities: string[] }
  ): Promise<number> {
    const response = await this.llmGateway.complete({
      model: 'gpt-4o-mini',
      messages: [{
        role: 'system',
        content: `Evaluate how well a partner can help achieve a goal. Score 0.0-1.0.
Output JSON: {"score": 0.85, "reasoning": "..."}`
      }, {
        role: 'user',
        content: `Goal: ${goal.description}
Goal Type: ${goal.type}

Partner: ${partner.name}
Industry: ${partner.industry}
Capabilities: ${partner.capabilities.join(', ')}

Can this partner help achieve the goal?`
      }],
      response_format: { type: 'json_object' },
      temperature: 0.3
    });

    const analysis = JSON.parse(response.choices[0].message.content);
    return analysis.score;
  }

  private calculateOverlap(list1: string[], list2: string[]): number {
    if (list1.length === 0 || list2.length === 0) return 0;
    
    const set1 = new Set(list1.map(s => s.toLowerCase()));
    const set2 = new Set(list2.map(s => s.toLowerCase()));
    
    const intersection = new Set([...set1].filter(x => set2.has(x)));
    return intersection.size / Math.min(set1.size, set2.size);
  }
}
```

### 4. Cultural Trend Momentum

```typescript
// packages/affinity-engine/src/components/cultural-momentum.ts

interface TrendSignal {
  trend_id: string;
  trend_name: string;
  momentum_score: number;  // 0-1
  emergence_date: Date;
  peak_prediction?: Date;
  social_velocity: number;  // Rate of mentions/engagement
  audience_adoption: number;  // % of target demo engaged
}

export class CulturalMomentumScorer {
  constructor(
    private neo4j: Neo4jClient,
    private socialListening: SocialListeningService
  ) {}

  async calculateMomentum(
    entityId: string,
    partnerEntityId: string
  ): Promise<number> {
    // 1. Find shared trends
    const sharedTrends = await this.findSharedTrends(entityId, partnerEntityId);

    if (sharedTrends.length === 0) return 0;

    // 2. Score each trend's current momentum
    const trendScores = await Promise.all(
      sharedTrends.map(trend => this.scoreTrendMomentum(trend))
    );

    // 3. Weight by entity alignment strength
    const weightedScores = await Promise.all(
      sharedTrends.map(async (trend, idx) => {
        const alignmentStrength = await this.getAlignmentStrength(
          entityId,
          partnerEntityId,
          trend.trend_id
        );
        return trendScores[idx] * alignmentStrength;
      })
    );

    return weightedScores.reduce((sum, score) => sum + score, 0) / weightedScores.length;
  }

  private async findSharedTrends(
    entity1: string,
    entity2: string
  ): Promise<TrendSignal[]> {
    const session = this.neo4j.session();
    
    try {
      const result = await session.run(`
        MATCH (e1 {id: $entity1})-[:ALIGNS_WITH]->(t:Trend)<-[:ALIGNS_WITH]-(e2 {id: $entity2})
        RETURN 
          t.id as trend_id,
          t.name as trend_name,
          t.momentum_score as momentum_score,
          t.emergence_date as emergence_date,
          t.peak_date as peak_prediction
        ORDER BY t.momentum_score DESC
      `, { entity1, entity2 });

      return result.records.map(record => ({
        trend_id: record.get('trend_id'),
        trend_name: record.get('trend_name'),
        momentum_score: record.get('momentum_score'),
        emergence_date: new Date(record.get('emergence_date')),
        peak_prediction: record.get('peak_prediction') 
          ? new Date(record.get('peak_prediction'))
          : undefined,
        social_velocity: 0,  // Will be enriched
        audience_adoption: 0
      }));
    } finally {
      await session.close();
    }
  }

  private async scoreTrendMomentum(trend: TrendSignal): Promise<number> {
    const now = new Date();
    
    // 1. Base momentum from Neo4j
    let score = trend.momentum_score;

    // 2. Lifecycle position (predictive: "skate to where puck is going")
    const daysSinceEmergence = differenceInDays(now, trend.emergence_date);
    
    if (daysSinceEmergence < 90) {
      // Emerging (0-3 months) - BEST TIME (predictive advantage)
      score *= 1.3;
    } else if (daysSinceEmergence < 180) {
      // Growing (3-6 months) - STILL GOOD
      score *= 1.1;
    } else if (!trend.peak_prediction || now < trend.peak_prediction) {
      // Mature but pre-peak - neutral
      score *= 1.0;
    } else {
      // Post-peak - declining relevance
      score *= 0.4;
    }

    // 3. Real-time social velocity
    if (this.socialListening) {
      const realtimeSignals = await this.socialListening.getTrendVelocity(trend.trend_name);
      score *= (1 + realtimeSignals.velocity_multiplier);
    }

    return Math.max(0, Math.min(1.5, score));  // Cap at 1.5 for exceptional trends
  }

  private async getAlignmentStrength(
    entity1: string,
    entity2: string,
    trendId: string
  ): Promise<number> {
    const session = this.neo4j.session();
    
    try {
      const result = await session.run(`
        MATCH (e1 {id: $entity1})-[r1:ALIGNS_WITH]->(t:Trend {id: $trendId})<-[r2:ALIGNS_WITH]-(e2 {id: $entity2})
        RETURN 
          r1.alignment_score as score1,
          r2.alignment_score as score2
      `, { entity1, entity2, trendId });

      if (result.records.length === 0) return 0.5;

      const record = result.records[0];
      const score1 = record.get('score1') || 0.5;
      const score2 = record.get('score2') || 0.5;

      // Average alignment strength
      return (score1 + score2) / 2;
    } finally {
      await session.close();
    }
  }
}
```

### 5. Historical Deal Performance (Feedback Loop)

```typescript
// packages/affinity-engine/src/components/historical-performance.ts

interface DealOutcome {
  deal_id: string;
  entity1_id: string;
  entity2_id: string;
  deal_value: number;
  success_score: number;  // 0-5 scale
  engagement_metrics: {
    impressions: number;
    engagement_rate: number;
    conversion_rate: number;
  };
  completed_at: Date;
  shared_affinities: string[];  // Trend IDs that were aligned
}

export class HistoricalPerformanceScorer {
  constructor(private db: Database, private neo4j: Neo4jClient) {}

  async calculateHistoricalScore(
    entity1: string,
    entity2: string
  ): Promise<number> {
    // 1. Find similar past deals
    const similarDeals = await this.findSimilarDeals(entity1, entity2);

    if (similarDeals.length === 0) return 0.5;  // Neutral if no history

    // 2. Calculate weighted success rate
    const weightedSuccess = this.calculateWeightedSuccess(similarDeals);

    // 3. Adjust for recency
    const recencyAdjusted = this.applyRecencyWeighting(weightedSuccess, similarDeals);

    return recencyAdjusted;
  }

  private async findSimilarDeals(
    entity1: string,
    entity2: string
  ): Promise<DealOutcome[]> {
    // Find deals involving either entity
    const deals = await this.db
      .selectFrom('partnership_outcomes')
      .leftJoin('partnership_affinities', 'partnership_outcomes.deal_id', 'partnership_affinities.deal_id')
      .select([
        'partnership_outcomes.deal_id',
        'partnership_outcomes.entity1_id',
        'partnership_outcomes.entity2_id',
        'partnership_outcomes.deal_value',
        'partnership_outcomes.success_score',
        'partnership_outcomes.engagement_metrics',
        'partnership_outcomes.completed_at',
        sql<string[]>`array_agg(partnership_affinities.trend_id)`.as('shared_affinities')
      ])
      .where((eb) =>
        eb.or([
          eb('entity1_id', '=', entity1),
          eb('entity1_id', '=', entity2),
          eb('entity2_id', '=', entity1),
          eb('entity2_id', '=', entity2)
        ])
      )
      .where('status', '=', 'completed')
      .groupBy(['deal_id', 'entity1_id', 'entity2_id', 'deal_value', 'success_score', 'engagement_metrics', 'completed_at'])
      .execute();

    // Filter by affinity pattern similarity
    const currentAffinities = await this.getCurrentAffinities(entity1, entity2);
    
    return deals.filter(deal => {
      const overlap = this.calculateAffinityOverlap(
        deal.shared_affinities,
        currentAffinities
      );
      return overlap > 0.5;  // At least 50% affinity overlap
    });
  }

  private calculateWeightedSuccess(deals: DealOutcome[]): number {
    if (deals.length === 0) return 0.5;

    // Weight by deal value (larger deals have more signal)
    const totalValue = deals.reduce((sum, d) => sum + d.deal_value, 0);
    
    const weightedSum = deals.reduce((sum, deal) => {
      const weight = deal.deal_value / totalValue;
      const normalizedScore = deal.success_score / 5.0;  // Normalize to 0-1
      return sum + (normalizedScore * weight);
    }, 0);

    return weightedSum;
  }

  private applyRecencyWeighting(
    baseScore: number,
    deals: DealOutcome[]
  ): number {
    const now = new Date();
    
    // Recent deals get higher confidence weight
    const recencyWeights = deals.map(deal => {
      const ageInDays = differenceInDays(now, deal.completed_at);
      return Math.exp(-ageInDays / 365);  // Exponential decay over 1 year
    });

    const avgRecency = recencyWeights.reduce((sum, w) => sum + w, 0) / recencyWeights.length;
    
    // Blend base score with confidence adjustment
    const confidence = Math.min(deals.length / 10, 1.0);  // Need 10+ deals for full confidence
    
    return baseScore * (0.7 + 0.3 * avgRecency * confidence);
  }

  private async getCurrentAffinities(entity1: string, entity2: string): Promise<string[]> {
    const session = this.neo4j.session();
    
    try {
      const result = await session.run(`
        MATCH (e1 {id: $entity1})-[:ALIGNS_WITH]->(t:Trend)<-[:ALIGNS_WITH]-(e2 {id: $entity2})
        RETURN collect(t.id) as trend_ids
      `, { entity1, entity2 });

      return result.records[0]?.get('trend_ids') || [];
    } finally {
      await session.close();
    }
  }

  private calculateAffinityOverlap(list1: string[], list2: string[]): number {
    if (list1.length === 0 || list2.length === 0) return 0;
    
    const set1 = new Set(list1);
    const set2 = new Set(list2);
    
    const intersection = new Set([...set1].filter(x => set2.has(x)));
    return intersection.size / Math.min(set1.size, set2.size);
  }
}
```

### 6. Authenticity Scoring

```typescript
// packages/affinity-engine/src/components/authenticity.ts

export class AuthenticityCalculator {
  constructor(
    private llmGateway: LLMGatewayClient,
    private db: Database,
    private neo4j: Neo4jClient
  ) {}

  async calculate(input: {
    entity1_id: string;
    entity1_type: string;
    entity2_id: string;
    entity2_type: string;
  }): Promise<number> {
    // Gather evidence for LLM analysis
    const evidence = await this.gatherEvidence(input);

    // Use LLM to assess authenticity
    const response = await this.llmGateway.complete({
      model: 'gpt-4o-mini',
      messages: [{
        role: 'system',
        content: `You are an expert at detecting authentic vs opportunistic partnerships.
Analyze the evidence and score authenticity from 0.0 to 1.0.
1.0 = genuinely aligned (values match actions, consistent history)
0.0 = opportunistic (jumping on trends, no historical basis)

Output JSON: {"score": 0.85, "reasoning": "...", "red_flags": [...]}`
      }, {
        role: 'user',
        content: this.formatEvidence(evidence)
      }],
      response_format: { type: 'json_object' },
      temperature: 0.3
    });

    const analysis = JSON.parse(response.choices[0].message.content);
    
    // Store analysis for explainability
    await this.storeAuthenticityAnalysis(input, analysis);

    return Math.max(0, Math.min(1, analysis.score));
  }

  private async gatherEvidence(input: any): Promise<any> {
    const [entity1Data, entity2Data, relationshipData] = await Promise.all([
      this.getEntityData(input.entity1_id),
      this.getEntityData(input.entity2_id),
      this.getRelationshipData(input.entity1_id, input.entity2_id)
    ]);

    return { entity1: entity1Data, entity2: entity2Data, relationship: relationshipData };
  }

  private formatEvidence(evidence: any): string {
    return `
# Entity 1: ${evidence.entity1.name}
- Stated Values: ${evidence.entity1.values?.join(', ') || 'Unknown'}
- Historical Actions: ${evidence.entity1.actions || 'Limited data'}
- Trend History: ${evidence.entity1.trend_history || 'New'}

# Entity 2: ${evidence.entity2.name}
- Description: ${evidence.entity2.description || 'Unknown'}
- Target Audience: ${evidence.entity2.audience || 'General'}

# Relationship Context
${evidence.relationship.description || 'No prior relationship'}

Assess: Is this alignment authentic or opportunistic?
    `.trim();
  }

  private async getEntityData(entityId: string): Promise<any> {
    // Query Neo4j for entity properties
    const session = this.neo4j.session();
    try {
      const result = await session.run(`
        MATCH (e {id: $entityId})
        OPTIONAL MATCH (e)-[:ALIGNS_WITH]->(t:Trend)
        RETURN e, collect(t.name) as trends
      `, { entityId });

      if (result.records.length === 0) return {};

      const record = result.records[0];
      const entity = record.get('e').properties;
      const trends = record.get('trends');

      return {
        name: entity.name,
        values: entity.values,
        actions: entity.historical_actions,
        trend_history: trends.join(', '),
        description: entity.description,
        audience: entity.target_audience
      };
    } finally {
      await session.close();
    }
  }

  private async getRelationshipData(entity1: string, entity2: string): Promise<any> {
    const session = this.neo4j.session();
    try {
      const result = await session.run(`
        MATCH (e1 {id: $entity1}), (e2 {id: $entity2})
        OPTIONAL MATCH path = (e1)-[*..3]-(e2)
        RETURN 
          CASE WHEN path IS NOT NULL 
            THEN 'Connected through ' + toString(length(path)) + ' degrees'
            ELSE 'No prior connection'
          END as description
      `, { entity1, entity2 });

      return {
        description: result.records[0]?.get('description') || 'Unknown'
      };
    } finally {
      await session.close();
    }
  }

  private async storeAuthenticityAnalysis(input: any, analysis: any): Promise<void> {
    await this.db.insertInto('authenticity_analyses')
      .values({
        entity1_id: input.entity1_id,
        entity2_id: input.entity2_id,
        score: analysis.score,
        reasoning: analysis.reasoning,
        red_flags: JSON.stringify(analysis.red_flags),
        analyzed_at: new Date()
      })
      .execute();
  }
}
```

### 7. Main Affinity Engine (Orchestration)

```typescript
// packages/affinity-engine/src/affinity-engine.ts

interface AffinityScoreRequest {
  user_input: UserInput;
  partner_entity_id: string;
  explain?: boolean;
}

interface AffinityScoreResponse {
  score: number;  // 0-100 scale
  confidence: number;
  components: {
    semantic: number;
    goal_alignment: number;
    cultural_momentum: number;
    historical: number;
    authenticity: number;
  };
  top_affinities: string[];  // Shared trends/themes
  recommendation: 'strong_match' | 'good_match' | 'moderate_match' | 'weak_match';
  explanation?: {
    strengths: string[];
    concerns: string[];
    timing_factors: string[];
  };
  processing_time_ms: number;
}

export class AffinityEngine {
  constructor(
    private tripleExtractor: TripleExtractor,
    private graphPopulator: KnowledgeGraphPopulator,
    private semanticScorer: SemanticSimilarityCalculator,
    private goalScorer: GoalAlignmentScorer,
    private momentumScorer: CulturalMomentumScorer,
    private historicalScorer: HistoricalPerformanceScorer,
    private authenticityScorer: AuthenticityCalculator,
    private cache: RedisCache
  ) {}

  async calculateAffinity(request: AffinityScoreRequest): Promise<AffinityScoreResponse> {
    const startTime = Date.now();

    // 1. Check cache for subsequent searches (1-2 second response)
    const cacheKey = `affinity:${hashUserInput(request.user_input)}:${request.partner_entity_id}`;
    const cached = await this.cache.get(cacheKey);
    if (cached) {
      const result = JSON.parse(cached);
      result.processing_time_ms = Date.now() - startTime;
      return result;
    }

    // 2. Extract triples (20-30 seconds for first-time users)
    const triples = await this.tripleExtractor.extractFromInput(request.user_input);
    
    // 3. Populate knowledge graph
    await this.graphPopulator.populateGraph(triples, request.user_input.user_id);

    // 4. Calculate component scores in parallel
    const userEntityId = this.extractUserEntityId(triples);
    const goals = this.extractGoals(request.user_input);

    const [semantic, goalAlignment, momentum, historical, authenticity] = await Promise.all([
      this.semanticScorer.calculate({
        entity1_id: userEntityId,
        entity1_type: 'content',
        entity2_id: request.partner_entity_id,
        entity2_type: 'brand'
      }),
      this.goalScorer.calculateAlignment(goals, request.partner_entity_id),
      this.momentumScorer.calculateMomentum(userEntityId, request.partner_entity_id),
      this.historicalScorer.calculateHistoricalScore(userEntityId, request.partner_entity_id),
      this.authenticityScorer.calculate({
        entity1_id: userEntityId,
        entity1_type: 'content',
        entity2_id: request.partner_entity_id,
        entity2_type: 'brand'
      })
    ]);

    // 5. Weighted combination (aligned with framework priorities)
    const weights = {
      semantic: 0.35,         // Semantic similarity
      goal_alignment: 0.30,   // Goal alignment (time-stamped)
      cultural_momentum: 0.20, // Cultural trends (predictive)
      historical: 0.10,       // Historical performance
      authenticity: 0.05      // Authenticity check
    };

    const finalScore = 
      semantic * weights.semantic +
      goalAlignment * weights.goal_alignment +
      momentum * weights.cultural_momentum +
      historical * weights.historical +
      authenticity * weights.authenticity;

    // 6. Convert to 0-100 scale
    const normalizedScore = finalScore * 100;

    // 7. Determine recommendation tier
    const recommendation = 
      normalizedScore >= 80 ? 'strong_match' :
      normalizedScore >= 65 ? 'good_match' :
      normalizedScore >= 50 ? 'moderate_match' : 'weak_match';

    // 8. Generate explanation if requested
    let explanation;
    if (request.explain) {
      explanation = await this.generateExplanation({
        semantic, goalAlignment, momentum, historical, authenticity
      }, weights, userEntityId, request.partner_entity_id);
    }

    // 9. Find shared affinities
    const topAffinities = await this.findTopSharedAffinities(
      userEntityId,
      request.partner_entity_id
    );

    const response: AffinityScoreResponse = {
      score: normalizedScore,
      confidence: this.calculateConfidence({ semantic, goalAlignment, momentum, historical, authenticity }),
      components: {
        semantic,
        goal_alignment: goalAlignment,
        cultural_momentum: momentum,
        historical,
        authenticity
      },
      top_affinities: topAffinities,
      recommendation,
      explanation,
      processing_time_ms: Date.now() - startTime
    };

    // 10. Cache for subsequent searches (90-day TTL with auto-refresh)
    await this.cache.setex(cacheKey, 86400 * 90, JSON.stringify(response));

    return response;
  }

  private extractUserEntityId(triples: ExtractedTriples): string {
    // Find primary entity mentioned by user (e.g., their show, brand, etc.)
    for (const triple of triples.facts) {
      if (triple.predicate === 'produces' || triple.predicate === 'owns') {
        return triple.object;  // "American Beauty Star"
      }
    }
    
    // Fallback: use first entity mentioned
    return triples.facts[0]?.subject || 'unknown';
  }

  private extractGoals(input: UserInput): Goal[] {
    // Parse "What do you want?" into structured goals
    // This would use LLM to extract structured goal data
    return [];  // Simplified for example
  }

  private calculateConfidence(components: Record<string, number>): number {
    // Higher confidence when all components agree
    const values = Object.values(components);
    const mean = values.reduce((sum, v) => sum + v, 0) / values.length;
    const variance = values.reduce((sum, v) => sum + Math.pow(v - mean, 2), 0) / values.length;
    const stdDev = Math.sqrt(variance);

    // Low variance = high confidence
    if (stdDev < 0.15) return 0.95;
    if (stdDev < 0.25) return 0.80;
    if (stdDev < 0.35) return 0.65;
    return 0.50;
  }

  private async generateExplanation(
    components: Record<string, number>,
    weights: Record<string, number>,
    entity1: string,
    entity2: string
  ): Promise<{ strengths: string[]; concerns: string[]; timing_factors: string[] }> {
    const strengths: string[] = [];
    const concerns: string[] = [];
    const timing_factors: string[] = [];

    // Identify strong components
    if (components.semantic > 0.75) {
      strengths.push("Strong semantic alignment - shared language and themes");
    }
    if (components.goal_alignment > 0.75) {
      strengths.push("Excellent goal alignment - partner advances your objectives");
    }
    if (components.cultural_momentum > 0.75) {
      timing_factors.push("Perfect timing - shared trends at peak momentum");
    }
    if (components.historical > 0.75) {
      strengths.push("Proven track record - similar partnerships succeeded");
    }

    // Identify concerns
    if (components.authenticity < 0.5) {
      concerns.push("Authenticity concern - partnership may appear opportunistic");
    }
    if (components.goal_alignment < 0.4) {
      concerns.push("Limited goal alignment - partner may not advance objectives");
    }
    if (components.cultural_momentum < 0.4) {
      timing_factors.push("Timing risk - shared trends declining");
    }

    return { strengths, concerns, timing_factors };
  }

  private async findTopSharedAffinities(
    entity1: string,
    entity2: string
  ): Promise<string[]> {
    const session = this.neo4j.session();
    
    try {
      const result = await session.run(`
        MATCH (e1 {id: $entity1})-[:ALIGNS_WITH]->(t:Trend)<-[:ALIGNS_WITH]-(e2 {id: $entity2})
        RETURN t.name as trend_name
        ORDER BY t.momentum_score DESC
        LIMIT 5
      `, { entity1, entity2 });

      return result.records.map(r => r.get('trend_name'));
    } finally {
      await session.close();
    }
  }
}
```

---

## Database Schema

```sql
-- Partnership outcomes (feedback loop)
CREATE TABLE partnership_outcomes (
  deal_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  entity1_id UUID NOT NULL,
  entity2_id UUID NOT NULL,
  deal_value DECIMAL(12,2),
  success_score DECIMAL(3,2),  -- 0.00-5.00
  engagement_metrics JSONB,
  status VARCHAR(50),  -- 'pending' | 'active' | 'completed' | 'cancelled'
  completed_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_partnership_outcomes_entities ON partnership_outcomes(entity1_id, entity2_id);
CREATE INDEX idx_partnership_outcomes_status ON partnership_outcomes(status);

-- Partnership affinities (which trends aligned)
CREATE TABLE partnership_affinities (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  deal_id UUID REFERENCES partnership_outcomes(deal_id),
  trend_id UUID NOT NULL,
  alignment_score DECIMAL(5,4),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_partnership_affinities_deal ON partnership_affinities(deal_id);
CREATE INDEX idx_partnership_affinities_trend ON partnership_affinities(trend_id);

-- Authenticity analyses (for explainability)
CREATE TABLE authenticity_analyses (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  entity1_id UUID NOT NULL,
  entity2_id UUID NOT NULL,
  score DECIMAL(5,4),
  reasoning TEXT,
  red_flags JSONB,
  analyzed_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_authenticity_analyses_entities ON authenticity_analyses(entity1_id, entity2_id);

-- Goal tracking (time-stamped first-party data)
CREATE TABLE member_goals (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  member_id UUID NOT NULL,
  goal_type VARCHAR(50),  -- 'market_expansion' | 'revenue_growth' | etc.
  description TEXT,
  target_value DECIMAL(12,2),
  target_demographics JSONB,
  target_geography JSONB,
  timeline TIMESTAMPTZ,
  timestamp TIMESTAMPTZ DEFAULT NOW(),  -- When goal was set/updated
  active BOOLEAN DEFAULT true
);

CREATE INDEX idx_member_goals_member ON member_goals(member_id);
CREATE INDEX idx_member_goals_timestamp ON member_goals(timestamp DESC);
```

---

## Performance Optimization

### Data Freshness Management

```typescript
// Auto-refresh stale triples (30-90 day TTL)
export class TripleFreshnessManager {
  async refreshStaleTriples(): Promise<void> {
    const session = this.neo4j.session();
    
    try {
      // Find triples older than 90 days
      const result = await session.run(`
        MATCH (n)
        WHERE n.last_updated < datetime() - duration({days: 90})
        RETURN n.id as entity_id, n.name as entity_name
        LIMIT 100
      `);

      // Enqueue for refresh
      for (const record of result.records) {
        await this.refreshQueue.add('refresh-entity', {
          entity_id: record.get('entity_id'),
          entity_name: record.get('entity_name')
        });
      }
    } finally {
      await session.close();
    }
  }
}
```

### Caching Strategy

```typescript
// Multi-level caching
const cacheConfig = {
  // Level 1: Full affinity scores (90-day TTL)
  affinityScores: {
    ttl: 86400 * 90,
    keyPattern: 'affinity:{user_input_hash}:{partner_id}'
  },
  
  // Level 2: Triple extractions (30-day TTL)
  tripleExtractions: {
    ttl: 86400 * 30,
    keyPattern: 'triples:{user_input_hash}'
  },
  
  // Level 3: Component scores (7-day TTL)
  componentScores: {
    ttl: 86400 * 7,
    keyPattern: 'component:{entity1}:{entity2}:{component_type}'
  }
};
```

---

## Migration Path

### Phase 1: Foundation (Months 1-3)
- Triple extraction from natural language
- Knowledge graph population
- Basic affinity scoring (semantic + goals)
- Public data enrichment (Wikipedia, IMDb)

### Phase 2: Intelligence (Months 4-6)
- Cultural trend monitoring
- Historical performance feedback
- Authenticity detection
- Social listening integration

### Phase 3: Scale (Months 7-9)
- Real-time triple updates
- Cross-industry expansion
- Advanced predictive analytics
- Deal outcome tracking

### Phase 4: Ecosystem (Months 10-12)
- API access for partners
- White-label solutions
- Multi-vertical knowledge graphs
- Advanced learning algorithms

---

## Alternatives Considered

### Alternative 1: Traditional CRM Matching
**Pros:** Familiar model, existing tools
**Cons:** No cultural intelligence, no semantic understanding, manual process
**Decision:** Affinity Engine provides unprecedented intelligence

### Alternative 2: Simple Keyword Matching
**Pros:** Fast, simple to implement
**Cons:** Misses semantic relationships, no authenticity detection, no predictive capability
**Decision:** Semantic triples capture deeper relationships

### Alternative 3: Pure LLM Matching
**Pros:** Flexible, natural language
**Cons:** Expensive, not real-time, no graph reasoning, hallucination risk
**Decision:** Hybrid approach balances intelligence with performance

---

## Consequences

### Positive
- Two-question simplicity ("Who are you?" + "What do you want?")
- 20-30 second first-time processing, 1-2 second subsequent
- Zero cold start via public data enrichment
- Self-improving through deal feedback loops
- Predictive cultural intelligence ("skate to where puck is going")
- Authentic partnership detection (genuine vs opportunistic)
- Scalable across all industries
- Data moat: proprietary affinity profiles

### Negative
- Complex system with multiple components
- Dependent on LLM quality for triple extraction
- Requires continuous public data ingestion
- Initial triple extraction latency (20-30s)

### Neutral
- Requires calibration of component weights
- Quality depends on deal outcome tracking
- Continuous monitoring needed for freshness

---

## Success Metrics

**Partnership Quality:**
- 60%+ partnership acceptance rate (vs 40% baseline)
- 75%+ partnership success rate (measured by completion)
- 80%+ client satisfaction with match quality

**Speed & Efficiency:**
- 20-30 seconds: First-time triple extraction
- 1-2 seconds: Subsequent searches (cached)
- 90%+ reduction in manual research time

**Business Impact:**
- Network effect: Each deal improves matches
- Data moat: Proprietary affinity intelligence
- Category leadership: Industry-first affinity profiles

**Technical Performance:**
- Scale to 1M+ entities
- 10M+ semantic triples
- 99.9% availability
- 90-day automatic freshness management

---

## References

- [TERAFFI Affinity Acceleration Framework™](../02-affinity-framework.md)
- [Affinity Engine Technical Specifications](../04-affinity-engine-technical.md)
- [Affinity Engine Proposal](../Affinity%20Engine%20Proposal.md)
- [Neo4j Graph Algorithms](https://neo4j.com/docs/graph-data-science/)
- [Semantic Triple Extraction](https://arxiv.org/abs/2004.02112)

---
