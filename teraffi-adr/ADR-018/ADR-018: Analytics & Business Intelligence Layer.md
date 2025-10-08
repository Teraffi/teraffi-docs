# ADR-018: Analytics & Business Intelligence Layer

**Status:** Proposed  
**Date:** 2025-10-01  
**Decision Makers:** Platform Architecture Team  
**Related ADRs:** ADR-007 (PostgreSQL), ADR-011 (Neo4j), ADR-015 (Affinity Scoring), ADR-017 (Deal Flow)

---

## Context

TERAFFI generates rich data across the platform: affinity scores, cultural trends, partnership outcomes, deal flow metrics, and engagement patterns. This data must be transformed into actionable business intelligence for multiple stakeholder types.

**User Personas:**

**Brand Executives:**
- "What's our partnership ROI compared to industry benchmarks?"
- "Which cultural trends should we invest in next quarter?"
- "How do our affinities compare to competitors?"

**IP Owners/Creators:**
- "What's my partnership success rate vs platform average?"
- "Which brands are most aligned with my audience?"
- "What deal structures yield the highest value?"

**TERAFFI Operations:**
- "What's our partnership conversion funnel?"
- "Which affinity components predict success best?"
- "Where are bottlenecks in deal flow?"
- "Cost per partnership, LTV per member"

**Investors/Board:**
- "Monthly recurring partnerships, growth rate"
- "Network effects: how does each partnership improve matching?"
- "Unit economics: CAC, LTV, payback period"

### Current Gap

Without analytics infrastructure:
- **No Visibility**: Data exists but not accessible
- **Manual Reports**: Hours of SQL queries and spreadsheets
- **Stale Data**: Reports run weekly/monthly, not real-time
- **No Self-Service**: Non-technical users can't explore data
- **Missing Insights**: Patterns hidden in raw data

### Requirements

**Functional:**
1. Data warehouse with dimensional modeling
2. ETL pipelines from PostgreSQL, Neo4j, external sources
3. Pre-built dashboards for each persona
4. Ad-hoc query interface for power users
5. Scheduled reports and alerts
6. API for programmatic access
7. Export capabilities (CSV, PDF, Excel)

**Non-Functional:**
1. Data freshness: <15 minutes lag for critical metrics
2. Query performance: <5 seconds for dashboard loads
3. Scalability: 100M+ events, 10+ years retention
4. Security: Row-level security based on tenant/role
5. Cost: <$500/month at moderate scale

---

## Decision

**Implement a modern analytics stack** with:
1. **Data Warehouse** (Azure Synapse or ClickHouse) for OLAP workloads
2. **ETL Pipeline** (dbt + Airflow) for transformations
3. **Visualization Layer** (Metabase or Superset) for dashboards
4. **Reverse ETL** for operational analytics back to product
5. **Dimensional Models** optimized for common queries

**Architecture:**
```
Operational Databases (PostgreSQL, Neo4j)
    ↓
Change Data Capture (Debezium) or Scheduled Extracts
    ↓
Staging Area (Raw data lake)
    ↓
dbt Transformations (dimension/fact tables)
    ↓
Data Warehouse (Synapse/ClickHouse)
    ↓
Visualization (Metabase/Superset)
    ↓
Dashboards, Reports, Alerts
```

---

## Rationale

### Why Data Warehouse Over Direct Queries

**OLTP vs OLAP:**
- PostgreSQL optimized for transactions (OLTP)
- Analytics requires aggregations across millions of rows (OLAP)
- Separate warehouse prevents analytics queries from degrading app performance

**Benefits:**
- Pre-aggregated metrics for fast queries
- Historical snapshots (slowly changing dimensions)
- Complex joins across data sources
- Schema optimized for analytics, not normalized for storage

### Technology Choices

**Azure Synapse Analytics:**
- **Pros**: Managed, scales well, Azure-native, SQL interface
- **Cons**: More expensive, vendor lock-in
- **Use case**: If already on Azure, need massive scale

**ClickHouse:**
- **Pros**: Open-source, extremely fast (columnar), cost-effective
- **Cons**: Self-hosted ops overhead, smaller community
- **Use case**: High-volume events, cost-sensitive, team has expertise

**Decision**: Start with ClickHouse for cost/performance, migrate to Synapse if needed

**Metabase vs Superset:**
- **Metabase**: Easier for non-technical users, better UX
- **Superset**: More powerful for data teams, extensive visualization types
- **Decision**: Metabase for simplicity (matches user base)

---

## Implementation Details

### 1. Dimensional Model Design

```sql
-- Core dimension tables

-- Date dimension (for time-based analysis)
CREATE TABLE dim_date (
  date_key INTEGER PRIMARY KEY,
  full_date DATE,
  year INTEGER,
  quarter INTEGER,
  month INTEGER,
  week INTEGER,
  day_of_week INTEGER,
  is_weekend BOOLEAN,
  fiscal_year INTEGER,
  fiscal_quarter INTEGER
);

-- Entity dimension (brands, members, content)
CREATE TABLE dim_entity (
  entity_key INTEGER PRIMARY KEY,
  entity_id UUID,
  entity_type VARCHAR(50),
  entity_name VARCHAR(255),
  industry VARCHAR(100),
  founded_year INTEGER,
  location VARCHAR(255),
  tier VARCHAR(50),  -- 'enterprise' | 'professional' | 'individual'
  tenant_id UUID,
  valid_from TIMESTAMP,
  valid_to TIMESTAMP,
  is_current BOOLEAN,
  -- Slowly changing dimension type 2
  version INTEGER
);

-- Trend dimension
CREATE TABLE dim_trend (
  trend_key INTEGER PRIMARY KEY,
  trend_id UUID,
  trend_name VARCHAR(255),
  category VARCHAR(50),
  lifecycle_stage VARCHAR(50),
  valid_from TIMESTAMP,
  valid_to TIMESTAMP,
  is_current BOOLEAN
);

-- Deal stage dimension
CREATE TABLE dim_deal_stage (
  stage_key INTEGER PRIMARY KEY,
  stage_name VARCHAR(50),
  stage_order INTEGER,
  stage_category VARCHAR(50)  -- 'discovery' | 'negotiation' | 'execution' | 'active'
);

-- Core fact tables

-- Affinity scores (periodic snapshots)
CREATE TABLE fact_affinity_scores (
  affinity_key INTEGER PRIMARY KEY,
  entity1_key INTEGER REFERENCES dim_entity(entity_key),
  entity2_key INTEGER REFERENCES dim_entity(entity_key),
  date_key INTEGER REFERENCES dim_date(date_key),
  
  -- Metrics
  affinity_score DECIMAL(5,2),
  semantic_score DECIMAL(5,4),
  goal_alignment_score DECIMAL(5,4),
  cultural_momentum_score DECIMAL(5,4),
  historical_score DECIMAL(5,4),
  authenticity_score DECIMAL(5,4),
  confidence_score DECIMAL(5,4),
  
  -- Dimensions
  recommendation_tier VARCHAR(20),
  
  computed_at TIMESTAMP
);

-- Partnership performance (accumulating snapshot)
CREATE TABLE fact_partnerships (
  partnership_key INTEGER PRIMARY KEY,
  partnership_id UUID,
  entity1_key INTEGER REFERENCES dim_entity(entity_key),
  entity2_key INTEGER REFERENCES dim_entity(entity_key),
  start_date_key INTEGER REFERENCES dim_date(date_key),
  end_date_key INTEGER REFERENCES dim_date(date_key),
  current_stage_key INTEGER REFERENCES dim_deal_stage(stage_key),
  
  -- Metrics
  deal_value DECIMAL(12,2),
  success_score DECIMAL(3,2),
  days_to_signed INTEGER,
  days_to_active INTEGER,
  milestone_completion_rate DECIMAL(5,4),
  
  -- Milestones achieved (accumulating)
  milestones_total INTEGER,
  milestones_completed INTEGER,
  
  -- Flags
  is_completed BOOLEAN,
  is_successful BOOLEAN,  -- success_score >= 3.5
  
  updated_at TIMESTAMP
);

-- Deal flow events (transaction fact)
CREATE TABLE fact_deal_events (
  event_key INTEGER PRIMARY KEY,
  partnership_key INTEGER REFERENCES fact_partnerships(partnership_key),
  event_date_key INTEGER REFERENCES dim_date(date_key),
  from_stage_key INTEGER REFERENCES dim_deal_stage(stage_key),
  to_stage_key INTEGER REFERENCES dim_deal_stage(stage_key),
  
  -- Metrics
  days_in_previous_stage INTEGER,
  actor_entity_key INTEGER REFERENCES dim_entity(entity_key),
  
  event_timestamp TIMESTAMP
);

-- Cultural trend performance (periodic snapshot)
CREATE TABLE fact_trend_metrics (
  metric_key INTEGER PRIMARY KEY,
  trend_key INTEGER REFERENCES dim_trend(trend_key),
  date_key INTEGER REFERENCES dim_date(date_key),
  
  -- Metrics
  momentum_score DECIMAL(5,4),
  velocity DECIMAL(5,4),
  signal_count INTEGER,
  engagement_total BIGINT,
  sentiment_avg DECIMAL(3,2),
  
  -- Entity alignment counts
  brands_aligned INTEGER,
  content_aligned INTEGER,
  
  measured_at TIMESTAMP
);

-- Usage analytics (transaction fact)
CREATE TABLE fact_user_activity (
  activity_key INTEGER PRIMARY KEY,
  user_entity_key INTEGER REFERENCES dim_entity(entity_key),
  activity_date_key INTEGER REFERENCES dim_date(date_key),
  
  -- Activity type
  activity_type VARCHAR(50),  -- 'search' | 'view_match' | 'contact' | 'proposal_view'
  
  -- Context
  target_entity_key INTEGER REFERENCES dim_entity(entity_key),
  partnership_key INTEGER REFERENCES fact_partnerships(partnership_key),
  
  activity_timestamp TIMESTAMP
);
```

### 2. ETL Pipeline with dbt

```yaml
# dbt/models/staging/stg_partnerships.sql
# Extract raw partnership data

WITH source AS (
  SELECT * FROM {{ source('teraffi', 'partnerships') }}
),

staged AS (
  SELECT
    id AS partnership_id,
    entity1_id,
    entity2_id,
    stage,
    partnership_type,
    deal_value,
    start_date,
    end_date,
    created_at,
    updated_at
  FROM source
)

SELECT * FROM staged
```

```yaml
# dbt/models/marts/fact_partnerships.sql
# Transform into fact table

WITH partnerships AS (
  SELECT * FROM {{ ref('stg_partnerships') }}
),

entities1 AS (
  SELECT * FROM {{ ref('dim_entity') }}
),

entities2 AS (
  SELECT * FROM {{ ref('dim_entity') }}
),

stages AS (
  SELECT * FROM {{ ref('dim_deal_stage') }}
),

milestones AS (
  SELECT
    deal_id,
    COUNT(*) AS milestones_total,
    COUNT(*) FILTER (WHERE completed = true) AS milestones_completed
  FROM {{ ref('stg_milestones') }}
  GROUP BY deal_id
),

joined AS (
  SELECT
    p.partnership_id,
    e1.entity_key AS entity1_key,
    e2.entity_key AS entity2_key,
    {{ date_to_key('p.start_date') }} AS start_date_key,
    {{ date_to_key('p.end_date') }} AS end_date_key,
    s.stage_key AS current_stage_key,
    p.deal_value,
    COALESCE(m.milestones_total, 0) AS milestones_total,
    COALESCE(m.milestones_completed, 0) AS milestones_completed,
    CASE 
      WHEN m.milestones_total > 0 
      THEN m.milestones_completed::DECIMAL / m.milestones_total 
      ELSE 0 
    END AS milestone_completion_rate,
    p.stage = 'completed' AS is_completed,
    p.updated_at
  FROM partnerships p
  LEFT JOIN entities1 e1 ON p.entity1_id = e1.entity_id AND e1.is_current = true
  LEFT JOIN entities2 e2 ON p.entity2_id = e2.entity_id AND e2.is_current = true
  LEFT JOIN stages s ON p.stage = s.stage_name
  LEFT JOIN milestones m ON p.partnership_id = m.deal_id
)

SELECT * FROM joined
```

```python
# airflow/dags/teraffi_analytics_etl.py
# Orchestration

from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.providers.dbt.operators.dbt import DbtRunOperator
from datetime import datetime, timedelta

default_args = {
    'owner': 'data-team',
    'retries': 2,
    'retry_delay': timedelta(minutes=5)
}

with DAG(
    'teraffi_analytics_etl',
    default_args=default_args,
    schedule_interval='0 * * * *',  # Hourly
    start_date=datetime(2025, 10, 1),
    catchup=False
) as dag:

    extract_postgres = PythonOperator(
        task_id='extract_postgres',
        python_callable=extract_from_postgres
    )

    extract_neo4j = PythonOperator(
        task_id='extract_neo4j',
        python_callable=extract_from_neo4j
    )

    dbt_run = DbtRunOperator(
        task_id='dbt_run',
        project_dir='/opt/dbt',
        profiles_dir='/opt/dbt',
        target='prod'
    )

    dbt_test = DbtRunOperator(
        task_id='dbt_test',
        project_dir='/opt/dbt',
        profiles_dir='/opt/dbt',
        target='prod',
        command='test'
    )

    [extract_postgres, extract_neo4j] >> dbt_run >> dbt_test
```

### 3. Pre-built Dashboards

```yaml
# metabase/dashboards/partnership_performance.yaml

dashboard:
  name: "Partnership Performance"
  description: "Track deal flow metrics and success rates"
  
  filters:
    - name: "date_range"
      type: "date"
      default: "last_30_days"
    - name: "entity"
      type: "entity_selector"
      
  cards:
    - title: "Partnership Funnel"
      visualization: "funnel"
      query: |
        SELECT 
          stage_name,
          COUNT(*) as partnerships
        FROM fact_partnerships fp
        JOIN dim_deal_stage ds ON fp.current_stage_key = ds.stage_key
        GROUP BY stage_name, stage_order
        ORDER BY stage_order
        
    - title: "Average Days to Signed"
      visualization: "number"
      query: |
        SELECT AVG(days_to_signed)
        FROM fact_partnerships
        WHERE is_completed = true
        
    - title: "Success Rate by Partnership Type"
      visualization: "bar"
      query: |
        SELECT 
          partnership_type,
          AVG(success_score) as avg_success
        FROM fact_partnerships
        WHERE is_completed = true
        GROUP BY partnership_type
        ORDER BY avg_success DESC
        
    - title: "Monthly Partnership Value"
      visualization: "line"
      query: |
        SELECT 
          d.year || '-' || LPAD(d.month::text, 2, '0') as month,
          SUM(fp.deal_value) as total_value,
          COUNT(*) as partnership_count
        FROM fact_partnerships fp
        JOIN dim_date d ON fp.start_date_key = d.date_key
        GROUP BY d.year, d.month
        ORDER BY d.year, d.month
        
    - title: "Top Performing Partnerships"
      visualization: "table"
      query: |
        SELECT
          e1.entity_name as entity1,
          e2.entity_name as entity2,
          fp.deal_value,
          fp.success_score,
          fp.milestone_completion_rate
        FROM fact_partnerships fp
        JOIN dim_entity e1 ON fp.entity1_key = e1.entity_key
        JOIN dim_entity e2 ON fp.entity2_key = e2.entity_key
        WHERE fp.is_completed = true
        ORDER BY fp.success_score DESC
        LIMIT 10
```

### 4. Analytics API

```typescript
// packages/analytics-api/src/routes/metrics.ts

import { FastifyPluginAsync } from 'fastify';

const metricsRoutes: FastifyPluginAsync = async (fastify) => {
  // Partnership funnel metrics
  fastify.get('/metrics/partnership-funnel', {
    schema: {
      querystring: {
        type: 'object',
        properties: {
          start_date: { type: 'string', format: 'date' },
          end_date: { type: 'string', format: 'date' },
          entity_id: { type: 'string', format: 'uuid' }
        }
      }
    }
  }, async (request, reply) => {
    const { start_date, end_date, entity_id } = request.query;

    const result = await fastify.clickhouse.query(`
      SELECT 
        stage_name,
        stage_order,
        COUNT(*) as count
      FROM fact_partnerships fp
      JOIN dim_deal_stage ds ON fp.current_stage_key = ds.stage_key
      WHERE 1=1
        ${start_date ? `AND fp.start_date >= '${start_date}'` : ''}
        ${end_date ? `AND fp.start_date <= '${end_date}'` : ''}
        ${entity_id ? `AND (fp.entity1_key = (SELECT entity_key FROM dim_entity WHERE entity_id = '${entity_id}' AND is_current = true)
                        OR fp.entity2_key = (SELECT entity_key FROM dim_entity WHERE entity_id = '${entity_id}' AND is_current = true))` : ''}
      GROUP BY stage_name, stage_order
      ORDER BY stage_order
    `);

    return result.data;
  });

  // Affinity score trends
  fastify.get('/metrics/affinity-trends', async (request, reply) => {
    const { entity_id, days = 90 } = request.query;

    const result = await fastify.clickhouse.query(`
      SELECT 
        d.full_date as date,
        AVG(fas.affinity_score) as avg_score,
        AVG(fas.confidence_score) as avg_confidence
      FROM fact_affinity_scores fas
      JOIN dim_date d ON fas.date_key = d.date_key
      WHERE fas.entity1_key = (SELECT entity_key FROM dim_entity WHERE entity_id = '${entity_id}' AND is_current = true)
        AND d.full_date >= CURRENT_DATE - INTERVAL ${days} DAY
      GROUP BY d.full_date
      ORDER BY d.full_date
    `);

    return result.data;
  });

  // Cultural trend performance
  fastify.get('/metrics/trend-performance', async (request, reply) => {
    const { trend_id } = request.query;

    const result = await fastify.clickhouse.query(`
      SELECT
        d.full_date as date,
        ftm.momentum_score,
        ftm.velocity,
        ftm.signal_count,
        ftm.brands_aligned
      FROM fact_trend_metrics ftm
      JOIN dim_trend dt ON ftm.trend_key = dt.trend_key
      JOIN dim_date d ON ftm.date_key = d.date_key
      WHERE dt.trend_id = '${trend_id}'
        AND dt.is_current = true
        AND d.full_date >= CURRENT_DATE - INTERVAL 90 DAY
      ORDER BY d.full_date
    `);

    return result.data;
  });
};

export default metricsRoutes;
```

### 5. Scheduled Reports

```typescript
// packages/analytics-api/src/reports/scheduled-reports.ts

export class ScheduledReportService {
  async generateMonthlyExecutiveReport(month: Date): Promise<void> {
    // Aggregate key metrics
    const metrics = await this.clickhouse.query(`
      WITH monthly_metrics AS (
        SELECT
          COUNT(*) FILTER (WHERE is_completed = true) as partnerships_completed,
          AVG(success_score) FILTER (WHERE is_completed = true) as avg_success_score,
          SUM(deal_value) as total_deal_value,
          AVG(days_to_signed) as avg_days_to_signed,
          COUNT(DISTINCT entity1_key) + COUNT(DISTINCT entity2_key) as active_entities
        FROM fact_partnerships fp
        JOIN dim_date d ON fp.start_date_key = d.date_key
        WHERE d.year = ${month.getFullYear()}
          AND d.month = ${month.getMonth() + 1}
      )
      SELECT * FROM monthly_metrics
    `);

    // Generate PDF report
    const report = await this.generatePDFReport({
      title: `TERAFFI Monthly Report - ${format(month, 'MMMM yyyy')}`,
      sections: [
        {
          title: 'Partnership Performance',
          metrics: metrics.data[0],
          charts: await this.generateCharts(month)
        },
        {
          title: 'Cultural Trends',
          data: await this.getTrendingSummary(month)
        },
        {
          title: 'Top Performers',
          data: await this.getTopPartners(month)
        }
      ]
    });

    // Email to stakeholders
    await this.emailService.send({
      to: 'executives@teraffi.io',
      subject: `Monthly Report - ${format(month, 'MMMM yyyy')}`,
      attachments: [{ filename: 'report.pdf', content: report }]
    });
  }

  async generateDealAlerts(): Promise<void> {
    // Find stalled deals (>30 days in same stage)
    const stalledDeals = await this.clickhouse.query(`
      SELECT
        fp.partnership_id,
        e1.entity_name as entity1,
        e2.entity_name as entity2,
        ds.stage_name,
        DATEDIFF(day, CURRENT_DATE, fp.updated_at) as days_stalled
      FROM fact_partnerships fp
      JOIN dim_entity e1 ON fp.entity1_key = e1.entity_key
      JOIN dim_entity e2 ON fp.entity2_key = e2.entity_key
      JOIN dim_deal_stage ds ON fp.current_stage_key = ds.stage_key
      WHERE fp.is_completed = false
        AND DATEDIFF(day, CURRENT_DATE, fp.updated_at) > 30
    `);

    for (const deal of stalledDeals.data) {
      await this.notificationService.send({
        type: 'deal_alert',
        priority: 'medium',
        title: 'Stalled Partnership',
        message: `Partnership between ${deal.entity1} and ${deal.entity2} has been in ${deal.stage_name} for ${deal.days_stalled} days`,
        action_url: `/deals/${deal.partnership_id}`
      });
    }
  }
}
```

---

## Database Schema (Analytics)

```sql
-- ClickHouse materialized views for real-time aggregations

CREATE MATERIALIZED VIEW mv_daily_partnership_metrics
ENGINE = SummingMergeTree()
PARTITION BY toYYYYMM(date)
ORDER BY (date, entity_id)
AS SELECT
  toDate(created_at) as date,
  entity1_id as entity_id,
  COUNT(*) as partnerships_created,
  SUM(deal_value) as total_value
FROM partnerships
GROUP BY date, entity_id;

-- Rollup for monthly aggregates
CREATE MATERIALIZED VIEW mv_monthly_metrics
ENGINE = SummingMergeTree()
PARTITION BY toYear(month)
ORDER BY (month)
AS SELECT
  toStartOfMonth(date) as month,
  SUM(partnerships_created) as total_partnerships,
  SUM(total_value) as total_value
FROM mv_daily_partnership_metrics
GROUP BY month;
```

---

## Migration Path

### Phase 1: Foundation (Months 1-2)
- Set up ClickHouse cluster
- Build dimensional models
- ETL pipeline for partnerships

### Phase 2: Dashboards (Months 3-4)
- Metabase deployment
- Pre-built dashboards (3-5 per persona)
- Scheduled reports

### Phase 3: Advanced (Months 5-6)
- Real-time streaming (Kafka → ClickHouse)
- ML-powered insights
- Predictive analytics

### Phase 4: Self-Service (Months 7-8)
- Ad-hoc query builder
- Embedded analytics in product
- API for custom integrations

---

## Success Metrics

**Adoption:**
- 80%+ of eligible users view dashboards weekly
- 50%+ create custom reports/queries
- <5 minutes average time to insight

**Performance:**
- <5 second dashboard load times
- <15 minute data freshness
- 99.9% query success rate

**Business Impact:**
- 40% reduction in time spent on manual reporting
- 3x increase in data-driven decisions
- Identified $500k+ in partnership optimization opportunities

---

## References

- [Dimensional Modeling (Kimball)](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/)
- [ClickHouse Documentation](https://clickhouse.com/docs)
- [dbt Best Practices](https://docs.getdbt.com/guides/best-practices)
- [Metabase Documentation](https://www.metabase.com/docs/latest/)

---
