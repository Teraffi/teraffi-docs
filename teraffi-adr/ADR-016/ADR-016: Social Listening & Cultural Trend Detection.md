# ADR-016: Social Listening & Cultural Trend Detection

**Status:** Proposed  
**Date:** 2025-10-01  
**Decision Makers:** Platform Architecture Team  
**Related ADRs:** ADR-011 (Neo4j), ADR-014 (LLM Gateway), ADR-015 (Affinity Scoring)

---

## Context

TERAFFI's Affinity Engine™ requires **real-time cultural intelligence** to "skate to where the puck is going" - identifying emerging trends before they peak. The system must monitor cultural conversations, detect shifts in consumer behavior, and track trend momentum to enable predictive partnership recommendations.

**Use Cases:**

**Emerging Trend Detection:**
- "Identify cultural movements gaining traction in the past 30 days"
- "Which trends are accelerating among Gen Z demographics?"
- "Detect shifts in brand perception and audience sentiment"

**Predictive Partnership Timing:**
- "Which trends should we align with NOW for maximum impact in 6 months?"
- "Alert when shared affinities between entities reach critical momentum"
- "Identify declining trends before they become liabilities"

**Cultural Context for Affinity Scoring:**
- "Is this trend authentic or manufactured/astroturfed?"
- "What's the velocity of conversation around this movement?"
- "Which demographics are adopting this trend and at what rate?"

### Current Gap

Without real-time cultural monitoring:
- **Reactive not Predictive**: Partnerships formed after trends peak (too late)
- **No Velocity Tracking**: Can't distinguish accelerating from decelerating trends
- **Limited Authenticity Detection**: Hard to spot opportunistic vs genuine alignments
- **Missing Demographic Signals**: Don't know which audiences are adopting trends

### Requirements

**Functional:**
1. Monitor social media, news, forums for cultural conversations
2. Extract emerging trends from unstructured text
3. Calculate trend momentum and velocity (rate of change)
4. Track demographic adoption patterns
5. Detect sentiment shifts around brands/trends
6. Alert when trends cross momentum thresholds
7. Feed trend data into Neo4j and Affinity Engine

**Non-Functional:**
1. Real-time processing: <5 minute latency from signal to database
2. Scale: Process 1M+ social signals daily
3. Accuracy: >80% precision on trend identification
4. Cost: <$500/month at moderate scale
5. Privacy: No PII collection, public data only

---

## Decision

**Implement a hybrid social listening pipeline** combining:
1. **Social Media APIs** (Twitter/X, Reddit, TikTok) for real-time signals
2. **News Aggregation** (RSS feeds, news APIs) for authoritative sources
3. **LLM-Based Trend Extraction** to identify patterns in unstructured text
4. **Time-Series Analysis** for momentum and velocity calculation
5. **Neo4j Integration** to populate trend nodes with cultural intelligence

**Architecture:**
```
Social Media APIs + News Sources
    ↓
Signal Collection (Streaming)
    ↓
Text Processing & Filtering
    ↓
LLM Trend Extraction
    ↓
Time-Series Analysis (Momentum & Velocity)
    ↓
Trend Scoring & Classification
    ↓
Neo4j Trend Nodes (Update/Create)
    ↓
Alert Engine (Threshold-based notifications)
```

---

## Rationale

### Why Hybrid Approach

**Social Media APIs:**
- **Pros**: Real-time signals, volume, demographic data
- **Cons**: API costs, rate limits, noise
- **Use**: Primary signal source for trending topics

**News Aggregation:**
- **Pros**: Authoritative, filtered, context-rich
- **Cons**: Slower, less volume
- **Use**: Validation and context for social signals

**LLM Trend Extraction:**
- **Pros**: Understands semantic relationships, identifies patterns
- **Cons**: Cost per analysis, latency
- **Use**: Convert unstructured text to structured trends

### Why This Matters for TERAFFI

**Competitive Advantage:**
- Predictive recommendations (not reactive)
- Early trend identification = better partnership timing
- Cultural authenticity detection (genuine vs opportunistic)

**Integration with Affinity Engine:**
- Trend momentum feeds into Cultural Momentum scoring (ADR-015)
- Demographic adoption data improves Goal Alignment
- Velocity metrics enable predictive matching

---

## Implementation Details

### 1. Social Media Signal Collection

```typescript
// packages/social-listening/src/collectors/twitter-collector.ts

interface SocialSignal {
  id: string;
  source: 'twitter' | 'reddit' | 'tiktok' | 'news';
  text: string;
  author_id?: string;
  engagement: {
    likes: number;
    shares: number;
    comments: number;
  };
  demographics?: {
    estimated_age_range?: string;
    location?: string;
  };
  timestamp: Date;
  urls?: string[];
  hashtags?: string[];
  mentions?: string[];
}

export class TwitterCollector {
  constructor(
    private twitterClient: TwitterApi,
    private signalQueue: BullQueue
  ) {}

  async collectTrendingTopics(): Promise<void> {
    // Use Twitter API v2 streaming endpoint
    const stream = await this.twitterClient.v2.searchStream({
      'tweet.fields': ['created_at', 'public_metrics', 'entities'],
      'user.fields': ['location', 'verified'],
      expansions: ['author_id']
    });

    stream.on('data', async (tweet) => {
      const signal: SocialSignal = {
        id: tweet.data.id,
        source: 'twitter',
        text: tweet.data.text,
        author_id: tweet.data.author_id,
        engagement: {
          likes: tweet.data.public_metrics.like_count,
          shares: tweet.data.public_metrics.retweet_count,
          comments: tweet.data.public_metrics.reply_count
        },
        timestamp: new Date(tweet.data.created_at),
        hashtags: tweet.data.entities?.hashtags?.map(h => h.tag),
        mentions: tweet.data.entities?.mentions?.map(m => m.username)
      };

      // Enqueue for processing
      await this.signalQueue.add('process-signal', signal);
    });
  }

  async searchKeywords(keywords: string[]): Promise<SocialSignal[]> {
    const signals: SocialSignal[] = [];
    
    for (const keyword of keywords) {
      const tweets = await this.twitterClient.v2.search(keyword, {
        max_results: 100,
        'tweet.fields': ['created_at', 'public_metrics'],
        start_time: subDays(new Date(), 7).toISOString()
      });

      for (const tweet of tweets.data) {
        signals.push({
          id: tweet.id,
          source: 'twitter',
          text: tweet.text,
          engagement: {
            likes: tweet.public_metrics.like_count,
            shares: tweet.public_metrics.retweet_count,
            comments: tweet.public_metrics.reply_count
          },
          timestamp: new Date(tweet.created_at)
        });
      }
    }

    return signals;
  }
}
```

```typescript
// packages/social-listening/src/collectors/reddit-collector.ts

export class RedditCollector {
  constructor(
    private redditClient: Snoowrap,
    private signalQueue: BullQueue
  ) {}

  async collectFromSubreddits(subreddits: string[]): Promise<void> {
    for (const subreddit of subreddits) {
      const posts = await this.redditClient
        .getSubreddit(subreddit)
        .getHot({ limit: 100 });

      for (const post of posts) {
        const signal: SocialSignal = {
          id: post.id,
          source: 'reddit',
          text: `${post.title}\n${post.selftext}`,
          engagement: {
            likes: post.ups,
            shares: 0,  // Reddit doesn't track shares
            comments: post.num_comments
          },
          timestamp: new Date(post.created_utc * 1000)
        };

        await this.signalQueue.add('process-signal', signal);
      }
    }
  }

  async searchTrending(timeframe: 'hour' | 'day' | 'week' = 'day'): Promise<SocialSignal[]> {
    const trendingPosts = await this.redditClient
      .getHot({ limit: 50, time: timeframe });

    return trendingPosts.map(post => ({
      id: post.id,
      source: 'reddit',
      text: `${post.title}\n${post.selftext}`,
      engagement: {
        likes: post.ups,
        shares: 0,
        comments: post.num_comments
      },
      timestamp: new Date(post.created_utc * 1000)
    }));
  }
}
```

### 2. LLM Trend Extraction

```typescript
// packages/social-listening/src/trend-extraction.ts

interface ExtractedTrend {
  name: string;
  category: 'lifestyle' | 'technology' | 'social' | 'economic' | 'cultural';
  keywords: string[];
  related_concepts: string[];
  demographic_appeal: string[];
  sentiment: 'positive' | 'neutral' | 'negative';
  confidence: number;
}

export class TrendExtractor {
  constructor(private llmGateway: LLMGatewayClient) {}

  async extractTrendsFromSignals(signals: SocialSignal[]): Promise<ExtractedTrend[]> {
    // Batch signals for efficiency
    const batchSize = 50;
    const batches = chunk(signals, batchSize);
    
    const allTrends: ExtractedTrend[] = [];

    for (const batch of batches) {
      const combinedText = batch.map(s => s.text).join('\n---\n');

      const response = await this.llmGateway.complete({
        model: 'gpt-4o-mini',  // Fast, cheap for trend extraction
        messages: [{
          role: 'system',
          content: `You are a cultural trend analyst. Identify emerging trends from social media signals.

Output JSON array of trends:
[
  {
    "name": "sustainable fashion",
    "category": "lifestyle",
    "keywords": ["eco-friendly", "circular economy", "upcycling"],
    "related_concepts": ["ethical consumption", "slow fashion"],
    "demographic_appeal": ["millennials", "gen_z"],
    "sentiment": "positive",
    "confidence": 0.85
  }
]

Only include trends mentioned multiple times or with strong engagement.`
        }, {
          role: 'user',
          content: `Analyze these social signals and extract emerging trends:\n\n${combinedText}`
        }],
        response_format: { type: 'json_object' },
        temperature: 0.3
      });

      const extracted = JSON.parse(response.choices[0].message.content);
      allTrends.push(...(extracted.trends || []));
    }

    // Deduplicate and merge similar trends
    return this.deduplicateTrends(allTrends);
  }

  private deduplicateTrends(trends: ExtractedTrend[]): ExtractedTrend[] {
    const trendMap = new Map<string, ExtractedTrend>();

    for (const trend of trends) {
      const key = trend.name.toLowerCase();
      
      if (trendMap.has(key)) {
        // Merge with existing
        const existing = trendMap.get(key)!;
        existing.keywords = [...new Set([...existing.keywords, ...trend.keywords])];
        existing.confidence = Math.max(existing.confidence, trend.confidence);
      } else {
        trendMap.set(key, trend);
      }
    }

    return Array.from(trendMap.values());
  }
}
```

### 3. Momentum & Velocity Calculation

```typescript
// packages/social-listening/src/momentum-calculator.ts

interface TrendTimeSeries {
  trend_id: string;
  trend_name: string;
  data_points: Array<{
    timestamp: Date;
    signal_count: number;
    engagement_total: number;
    sentiment_avg: number;
  }>;
}

interface MomentumMetrics {
  momentum_score: number;  // 0-1, current strength
  velocity: number;        // Rate of change (signals per day change)
  acceleration: number;    // Rate of velocity change
  lifecycle_stage: 'emerging' | 'growing' | 'peak' | 'declining';
  predicted_peak_date?: Date;
}

export class MomentumCalculator {
  async calculateMomentum(timeSeries: TrendTimeSeries): Promise<MomentumMetrics> {
    if (timeSeries.data_points.length < 7) {
      // Need at least 7 days of data
      return this.defaultMomentum();
    }

    // Sort by timestamp
    const sorted = timeSeries.data_points.sort((a, b) => 
      a.timestamp.getTime() - b.timestamp.getTime()
    );

    // Calculate daily signal counts
    const dailyCounts = this.aggregateByDay(sorted);

    // 1. Momentum Score (normalized signal count)
    const recentCount = dailyCounts.slice(-7).reduce((sum, d) => sum + d.count, 0);
    const maxHistorical = Math.max(...dailyCounts.map(d => d.count));
    const momentum_score = Math.min(recentCount / (maxHistorical * 7), 1.0);

    // 2. Velocity (rate of change in signals per day)
    const recentAvg = recentCount / 7;
    const previousWeekAvg = dailyCounts.slice(-14, -7).reduce((sum, d) => sum + d.count, 0) / 7;
    const velocity = (recentAvg - previousWeekAvg) / previousWeekAvg;

    // 3. Acceleration (rate of velocity change)
    const velocityHistory = this.calculateVelocityHistory(dailyCounts);
    const acceleration = velocityHistory.length >= 2
      ? (velocityHistory[velocityHistory.length - 1] - velocityHistory[velocityHistory.length - 2])
      : 0;

    // 4. Lifecycle Stage
    const lifecycle_stage = this.determineLifecycleStage(momentum_score, velocity, acceleration);

    // 5. Predicted Peak (if still growing)
    const predicted_peak_date = lifecycle_stage === 'growing' || lifecycle_stage === 'emerging'
      ? this.predictPeakDate(dailyCounts, velocity)
      : undefined;

    return {
      momentum_score,
      velocity,
      acceleration,
      lifecycle_stage,
      predicted_peak_date
    };
  }

  private aggregateByDay(dataPoints: TrendTimeSeries['data_points']): Array<{ date: Date; count: number }> {
    const dailyMap = new Map<string, number>();

    for (const point of dataPoints) {
      const dateKey = format(point.timestamp, 'yyyy-MM-dd');
      dailyMap.set(dateKey, (dailyMap.get(dateKey) || 0) + point.signal_count);
    }

    return Array.from(dailyMap.entries())
      .map(([date, count]) => ({ date: new Date(date), count }))
      .sort((a, b) => a.date.getTime() - b.date.getTime());
  }

  private determineLifecycleStage(
    momentum: number,
    velocity: number,
    acceleration: number
  ): MomentumMetrics['lifecycle_stage'] {
    if (momentum < 0.3 && velocity > 0.5 && acceleration > 0) {
      return 'emerging';  // Low momentum but accelerating fast
    }
    
    if (momentum >= 0.3 && momentum < 0.8 && velocity > 0) {
      return 'growing';  // Building momentum
    }
    
    if (momentum >= 0.8 || (velocity < 0.1 && velocity > -0.1)) {
      return 'peak';  // High momentum, velocity near zero
    }
    
    return 'declining';  // Negative velocity
  }

  private predictPeakDate(
    dailyCounts: Array<{ date: Date; count: number }>,
    velocity: number
  ): Date | undefined {
    if (velocity <= 0) return undefined;

    // Simple linear projection
    const recentCount = dailyCounts[dailyCounts.length - 1].count;
    const maxPossible = Math.max(...dailyCounts.map(d => d.count)) * 1.5;
    
    const daysToMax = (maxPossible - recentCount) / velocity;
    
    if (daysToMax > 0 && daysToMax < 180) {  // Reasonable range
      return addDays(new Date(), Math.ceil(daysToMax));
    }

    return undefined;
  }

  private calculateVelocityHistory(
    dailyCounts: Array<{ date: Date; count: number }>
  ): number[] {
    const velocities: number[] = [];
    const windowSize = 7;

    for (let i = windowSize; i < dailyCounts.length; i++) {
      const current = dailyCounts.slice(i - windowSize, i).reduce((sum, d) => sum + d.count, 0) / windowSize;
      const previous = dailyCounts.slice(i - windowSize * 2, i - windowSize).reduce((sum, d) => sum + d.count, 0) / windowSize;
      
      velocities.push((current - previous) / previous);
    }

    return velocities;
  }

  private defaultMomentum(): MomentumMetrics {
    return {
      momentum_score: 0.5,
      velocity: 0,
      acceleration: 0,
      lifecycle_stage: 'emerging'
    };
  }
}
```

### 4. Neo4j Trend Integration

```typescript
// packages/social-listening/src/trend-updater.ts

export class TrendUpdater {
  constructor(private neo4j: Neo4jClient) {}

  async updateOrCreateTrend(
    trend: ExtractedTrend,
    momentum: MomentumMetrics,
    signals: SocialSignal[]
  ): Promise<void> {
    const session = this.neo4j.session();

    try {
      // Calculate emergence date (first signal timestamp)
      const emergenceDate = signals.length > 0
        ? new Date(Math.min(...signals.map(s => s.timestamp.getTime())))
        : new Date();

      await session.run(`
        MERGE (t:Trend {name: $name})
        SET t.id = COALESCE(t.id, $trendId),
            t.category = $category,
            t.momentum_score = $momentumScore,
            t.velocity = $velocity,
            t.lifecycle_stage = $lifecycleStage,
            t.emergence_date = COALESCE(t.emergence_date, datetime($emergenceDate)),
            t.peak_date = CASE 
              WHEN $predictedPeakDate IS NOT NULL 
              THEN datetime($predictedPeakDate)
              ELSE t.peak_date 
            END,
            t.related_keywords = $keywords,
            t.demographic_appeal = $demographics,
            t.sentiment = $sentiment,
            t.last_updated = datetime(),
            t.signal_count = COALESCE(t.signal_count, 0) + $signalCount
      `, {
        trendId: generateId('trend_'),
        name: trend.name,
        category: trend.category,
        momentumScore: momentum.momentum_score,
        velocity: momentum.velocity,
        lifecycleStage: momentum.lifecycle_stage,
        emergenceDate: emergenceDate.toISOString(),
        predictedPeakDate: momentum.predicted_peak_date?.toISOString() || null,
        keywords: trend.keywords,
        demographics: trend.demographic_appeal,
        sentiment: trend.sentiment,
        signalCount: signals.length
      });

      // Create relationships to demographics
      for (const demo of trend.demographic_appeal) {
        await session.run(`
          MATCH (t:Trend {name: $trendName})
          MERGE (d:Demographic {segment: $demographic})
          MERGE (t)-[r:APPEALS_TO]->(d)
          SET r.strength = $momentum,
              r.updated_at = datetime()
        `, {
          trendName: trend.name,
          demographic: demo,
          momentum: momentum.momentum_score
        });
      }
    } finally {
      await session.close();
    }
  }

  async archiveDecliningTrends(): Promise<void> {
    const session = this.neo4j.session();

    try {
      // Archive trends that have been declining for 90+ days
      await session.run(`
        MATCH (t:Trend)
        WHERE t.lifecycle_stage = 'declining'
          AND t.last_updated < datetime() - duration({days: 90})
        SET t.archived = true,
            t.archived_at = datetime()
      `);
    } finally {
      await session.close();
    }
  }
}
```

### 5. Alert Engine

```typescript
// packages/social-listening/src/alert-engine.ts

interface TrendAlert {
  type: 'emerging' | 'accelerating' | 'peak' | 'declining';
  trend_name: string;
  momentum_score: number;
  velocity: number;
  message: string;
  affected_entities: string[];  // Entity IDs that align with this trend
  priority: 'high' | 'medium' | 'low';
}

export class AlertEngine {
  constructor(
    private neo4j: Neo4jClient,
    private notificationService: NotificationService
  ) {}

  async checkAlertsAndNotify(): Promise<void> {
    const alerts: TrendAlert[] = [];

    // 1. Emerging trends (momentum > 0.3, lifecycle = emerging)
    alerts.push(...await this.findEmergingTrends());

    // 2. Accelerating trends (velocity increase > 50%)
    alerts.push(...await this.findAcceleratingTrends());

    // 3. Peak warnings (predicted peak in next 30 days)
    alerts.push(...await this.findPeakWarnings());

    // 4. Declining trends (velocity negative, entities should know)
    alerts.push(...await this.findDecliningTrends());

    // Send notifications
    for (const alert of alerts) {
      await this.notifyAffectedEntities(alert);
    }
  }

  private async findEmergingTrends(): Promise<TrendAlert[]> {
    const session = this.neo4j.session();
    const alerts: TrendAlert[] = [];

    try {
      const result = await session.run(`
        MATCH (t:Trend)
        WHERE t.lifecycle_stage = 'emerging'
          AND t.momentum_score > 0.3
          AND t.last_updated > datetime() - duration({hours: 24})
        OPTIONAL MATCH (t)<-[:ALIGNS_WITH]-(entity)
        RETURN t.name as trend_name,
               t.momentum_score as momentum,
               t.velocity as velocity,
               collect(DISTINCT entity.id) as affected_entities
      `);

      for (const record of result.records) {
        alerts.push({
          type: 'emerging',
          trend_name: record.get('trend_name'),
          momentum_score: record.get('momentum'),
          velocity: record.get('velocity'),
          message: `Emerging trend "${record.get('trend_name')}" gaining traction (momentum: ${(record.get('momentum') * 100).toFixed(0)}%)`,
          affected_entities: record.get('affected_entities'),
          priority: 'high'
        });
      }
    } finally {
      await session.close();
    }

    return alerts;
  }

  private async findAcceleratingTrends(): Promise<TrendAlert[]> {
    const session = this.neo4j.session();
    const alerts: TrendAlert[] = [];

    try {
      const result = await session.run(`
        MATCH (t:Trend)
        WHERE t.velocity > 0.5  // 50%+ growth
          AND t.lifecycle_stage IN ['emerging', 'growing']
        OPTIONAL MATCH (t)<-[:ALIGNS_WITH]-(entity)
        RETURN t.name as trend_name,
               t.momentum_score as momentum,
               t.velocity as velocity,
               collect(DISTINCT entity.id) as affected_entities
      `);

      for (const record of result.records) {
        alerts.push({
          type: 'accelerating',
          trend_name: record.get('trend_name'),
          momentum_score: record.get('momentum'),
          velocity: record.get('velocity'),
          message: `Trend "${record.get('trend_name')}" accelerating rapidly (velocity: ${(record.get('velocity') * 100).toFixed(0)}%)`,
          affected_entities: record.get('affected_entities'),
          priority: 'high'
        });
      }
    } finally {
      await session.close();
    }

    return alerts;
  }

  private async notifyAffectedEntities(alert: TrendAlert): Promise<void> {
    for (const entityId of alert.affected_entities) {
      await this.notificationService.send({
        recipient_id: entityId,
        type: 'trend_alert',
        priority: alert.priority,
        title: `Cultural Trend Alert: ${alert.trend_name}`,
        message: alert.message,
        action_url: `/trends/${encodeURIComponent(alert.trend_name)}`
      });
    }
  }
}
```

---

## Database Schema

```sql
-- Time series data for trend momentum tracking
CREATE TABLE trend_signals (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  trend_name VARCHAR(255) NOT NULL,
  source VARCHAR(50),  -- 'twitter' | 'reddit' | 'news'
  signal_count INTEGER DEFAULT 1,
  engagement_total BIGINT DEFAULT 0,
  sentiment_avg DECIMAL(3,2),  -- -1.0 to 1.0
  timestamp TIMESTAMPTZ NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
) PARTITION BY RANGE (timestamp);

CREATE INDEX idx_trend_signals_trend ON trend_signals(trend_name, timestamp DESC);
CREATE INDEX idx_trend_signals_timestamp ON trend_signals(timestamp DESC);

-- Partitions by month for efficient querying
CREATE TABLE trend_signals_2025_10 PARTITION OF trend_signals
  FOR VALUES FROM ('2025-10-01') TO ('2025-11-01');

-- Alert history
CREATE TABLE trend_alerts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  alert_type VARCHAR(50),
  trend_name VARCHAR(255),
  momentum_score DECIMAL(5,4),
  velocity DECIMAL(5,4),
  message TEXT,
  affected_entity_ids UUID[],
  priority VARCHAR(20),
  sent_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_trend_alerts_trend ON trend_alerts(trend_name);
CREATE INDEX idx_trend_alerts_sent ON trend_alerts(sent_at DESC);
```

---

## Cost Management

```typescript
// Estimated costs at moderate scale:

// Twitter API (Basic tier): $100/month
// - 10,000 tweets/month
// - Search API access

// Reddit API: Free
// - Rate limited but sufficient for trending topics

// LLM Processing:
// - 1M signals/month
// - Batch 50 signals per request
// - 20k requests × $0.0001 = $2/month (GPT-4o-mini)

// Storage (PostgreSQL time series):
// - 1M signals/month
// - ~100 bytes per signal
// - 100MB/month negligible cost

// Total: ~$102/month at moderate scale
// Scales linearly with signal volume
```

---

## Migration Path

### Phase 1: Foundation (Months 1-2)
- Twitter and Reddit collectors
- Basic LLM trend extraction
- Simple momentum calculation
- Neo4j trend node updates

### Phase 2: Intelligence (Months 3-4)
- Time-series analysis
- Velocity and acceleration metrics
- Lifecycle stage detection
- Alert engine

### Phase 3: Scale (Months 5-6)
- Additional data sources (TikTok, news)
- Real-time streaming pipelines
- Advanced sentiment analysis
- Demographic adoption tracking

### Phase 4: Predictive (Months 7-9)
- Peak date prediction
- Trend correlation analysis
- Authenticity detection (manufactured vs organic)
- Cross-trend pattern recognition

---

## Alternatives Considered

### Alternative 1: Enterprise Social Listening Tools (Brandwatch, Sprinklr)
**Pros:** Comprehensive, established
**Cons:** Expensive ($2k-10k/month), limited customization, can't integrate directly with our graph
**Decision:** Build custom for tight integration and cost control

### Alternative 2: Simple Keyword Tracking
**Pros:** Simple, cheap
**Cons:** Misses emerging trends, no semantic understanding, no momentum tracking
**Decision:** Need sophisticated analysis for predictive intelligence

### Alternative 3: Pure LLM Analysis (No Social Data)
**Pros:** Simple architecture
**Cons:** Expensive, not real-time, hallucination risk, no velocity metrics
**Decision:** Need actual social signals for ground truth

---

## Consequences

### Positive
- Real-time cultural intelligence ("skate to where puck is going")
- Predictive partnership timing (before trends peak)
- Authenticity detection (organic vs manufactured)
- Velocity metrics enable better affinity scoring
- Competitive moat: proprietary cultural intelligence

### Negative
- API dependencies (Twitter, Reddit)
- Ongoing data processing costs
- Noise in social signals requires filtering
- Rate limits may constrain real-time capabilities

### Neutral
- Requires continuous calibration
- Quality depends on source diversity
- Privacy considerations (public data only)

---

## Success Metrics

**Detection Quality:**
- >80% precision on trend identification
- <5% false positive rate (noise filtered)
- Detect emerging trends 30-60 days before peak

**Performance:**
- <5 minute latency (signal to database)
- Process 1M+ signals daily
- <$500/month cost at moderate scale

**Business Impact:**
- 20%+ improvement in partnership timing
- 30%+ increase in early trend adoption
- Reduce failed partnerships due to declining trends

---

## References

- [Twitter API v2 Documentation](https://developer.twitter.com/en/docs/twitter-api)
- [Reddit API Documentation](https://www.reddit.com/dev/api)
- [Time Series Analysis for Trend Detection](https://otexts.com/fpp3/)
- [Social Media Analytics Best Practices](https://www.socialmediaexaminer.com/social-media-analytics/)

---
