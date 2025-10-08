# TERAFFI Database Schema
## Multi-Database Architecture for Partnership Network Platform

---

## Database Architecture Overview

TERAFFI utilizes a multi-database architecture optimized for different data types and access patterns:

- **PostgreSQL**: Primary relational data, transactions, user management
- **Neo4j**: Graph relationships for Affinity Engine™
- **Redis**: Caching, sessions, real-time features
- **Pinecone**: Vector embeddings for AI-powered matching
- **S3**: File storage for creative assets

---

## PostgreSQL Schema

### Core Member Tables

```sql
-- Members table: Core member information
CREATE TABLE members (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    email_verified BOOLEAN DEFAULT FALSE,
    subscription_tier subscription_tier_enum DEFAULT 'basic',
    status member_status_enum DEFAULT 'active',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    -- Indexes
    CONSTRAINT members_email_key UNIQUE (email)
);

CREATE INDEX idx_members_subscription_tier ON members (subscription_tier);
CREATE INDEX idx_members_status ON members (status);
CREATE INDEX idx_members_created_at ON members (created_at);

-- Member profiles: Detailed profile information
CREATE TABLE member_profiles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    member_id UUID NOT NULL REFERENCES members(id) ON DELETE CASCADE,
    
    -- Personal Information
    name VARCHAR(255) NOT NULL,
    title VARCHAR(255),
    company VARCHAR(255),
    industry VARCHAR(100),
    location VARCHAR(255),
    bio TEXT,
    avatar_url VARCHAR(500),
    
    -- Business Information
    company_size company_size_enum,
    annual_revenue revenue_range_enum,
    target_markets TEXT[], -- Array of target market strings
    brand_values TEXT[], -- Array of brand value strings
    partnership_types partnership_type_enum[],
    
    -- Contact Information
    phone VARCHAR(50),
    website VARCHAR(500),
    linkedin_url VARCHAR(500),
    
    -- Settings
    profile_visibility visibility_enum DEFAULT 'network',
    contact_preferences JSONB DEFAULT '{}',
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    CONSTRAINT member_profiles_member_id_key UNIQUE (member_id)
);

-- Member goals: Time-stamped business objectives
CREATE TABLE member_goals (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    member_id UUID NOT NULL REFERENCES members(id) ON DELETE CASCADE,
    type goal_type_enum NOT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    target_date DATE,
    priority priority_enum DEFAULT 'medium',
    status goal_status_enum DEFAULT 'active',
    
    -- Metadata
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_member_goals_member_id ON member_goals (member_id);
CREATE INDEX idx_member_goals_status ON member_goals (status);
CREATE INDEX idx_member_goals_priority ON member_goals (priority);

-- Member verification: Identity and business verification
CREATE TABLE member_verifications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    member_id UUID NOT NULL REFERENCES members(id) ON DELETE CASCADE,
    verification_type verification_type_enum NOT NULL,
    status verification_status_enum DEFAULT 'pending',
    documents JSONB, -- Store document URLs and metadata
    verified_by UUID REFERENCES members(id),
    verified_at TIMESTAMP WITH TIME ZONE,
    notes TEXT,
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_member_verifications_member_id ON member_verifications (member_id);
CREATE INDEX idx_member_verifications_status ON member_verifications (status);
```

### Authentication & Sessions

```sql
-- Authentication providers: OAuth and social login tracking
CREATE TABLE auth_providers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    member_id UUID NOT NULL REFERENCES members(id) ON DELETE CASCADE,
    provider auth_provider_enum NOT NULL,
    external_id VARCHAR(255) NOT NULL,
    access_token TEXT,
    refresh_token TEXT,
    expires_at TIMESTAMP WITH TIME ZONE,
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    CONSTRAINT auth_providers_unique UNIQUE (provider, external_id)
);

-- Sessions: User session management
CREATE TABLE user_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    member_id UUID NOT NULL REFERENCES members(id) ON DELETE CASCADE,
    session_token VARCHAR(512) UNIQUE NOT NULL,
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    ip_address INET,
    user_agent TEXT,
    is_active BOOLEAN DEFAULT TRUE,
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_user_sessions_member_id ON user_sessions (member_id);
CREATE INDEX idx_user_sessions_expires_at ON user_sessions (expires_at);
CREATE INDEX idx_user_sessions_session_token ON user_sessions (session_token);
```

### Partnership Management

```sql
-- Partnerships: Core partnership information
CREATE TABLE partnerships (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    partnership_type partnership_type_enum NOT NULL,
    status partnership_status_enum DEFAULT 'pending',
    stage partnership_stage_enum DEFAULT 'discovery',
    
    -- Financial Information
    total_value DECIMAL(12,2),
    currency currency_enum DEFAULT 'USD',
    payment_structure payment_structure_enum DEFAULT 'milestone_based',
    commission_rate DECIMAL(5,4) DEFAULT 0.08, -- 8% default
    
    -- Timeline
    start_date TIMESTAMP WITH TIME ZONE,
    end_date TIMESTAMP WITH TIME ZONE,
    duration_days INTEGER,
    
    -- Goals and Success Metrics
    goals TEXT[],
    success_metrics JSONB,
    
    -- Metadata
    created_by UUID NOT NULL REFERENCES members(id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_partnerships_status ON partnerships (status);
CREATE INDEX idx_partnerships_stage ON partnerships (stage);
CREATE INDEX idx_partnerships_created_by ON partnerships (created_by);
CREATE INDEX idx_partnerships_start_date ON partnerships (start_date);

-- Partnership members: Associates members with partnerships
CREATE TABLE partnership_members (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    partnership_id UUID NOT NULL REFERENCES partnerships(id) ON DELETE CASCADE,
    member_id UUID NOT NULL REFERENCES members(id) ON DELETE CASCADE,
    role partnership_role_enum NOT NULL,
    permissions partnership_permission_enum[] DEFAULT '{}',
    
    -- Contact Information
    contact_person VARCHAR(255),
    contact_email VARCHAR(255),
    contact_phone VARCHAR(50),
    
    -- Participation
    joined_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    left_at TIMESTAMP WITH TIME ZONE,
    is_active BOOLEAN DEFAULT TRUE,
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    CONSTRAINT partnership_members_unique UNIQUE (partnership_id, member_id)
);

CREATE INDEX idx_partnership_members_partnership_id ON partnership_members (partnership_id);
CREATE INDEX idx_partnership_members_member_id ON partnership_members (member_id);

-- Partnership milestones: Project milestones and deliverables
CREATE TABLE partnership_milestones (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    partnership_id UUID NOT NULL REFERENCES partnerships(id) ON DELETE CASCADE,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    
    -- Scheduling
    due_date TIMESTAMP WITH TIME ZONE,
    completed_date TIMESTAMP WITH TIME ZONE,
    status milestone_status_enum DEFAULT 'pending',
    
    -- Financial
    value DECIMAL(12,2),
    payment_status payment_status_enum DEFAULT 'pending',
    
    -- Deliverables
    deliverables JSONB DEFAULT '[]',
    approval_required BOOLEAN DEFAULT TRUE,
    approved_by UUID REFERENCES members(id),
    approved_at TIMESTAMP WITH TIME ZONE,
    
    -- Metadata
    created_by UUID NOT NULL REFERENCES members(id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_partnership_milestones_partnership_id ON partnership_milestones (partnership_id);
CREATE INDEX idx_partnership_milestones_due_date ON partnership_milestones (due_date);
CREATE INDEX idx_partnership_milestones_status ON partnership_milestones (status);
```

### Financial Management

```sql
-- Transactions: Financial transaction tracking
CREATE TABLE transactions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    partnership_id UUID REFERENCES partnerships(id),
    milestone_id UUID REFERENCES partnership_milestones(id),
    
    -- Transaction Details
    type transaction_type_enum NOT NULL,
    amount DECIMAL(12,2) NOT NULL,
    currency currency_enum DEFAULT 'USD',
    description TEXT,
    
    -- Status and Processing
    status transaction_status_enum DEFAULT 'pending',
    payment_method payment_method_enum,
    external_transaction_id VARCHAR(255), -- Stripe, PayPal, etc.
    
    -- Commission Calculation
    commission_rate DECIMAL(5,4),
    commission_amount DECIMAL(12,2),
    commission_status commission_status_enum DEFAULT 'pending',
    
    -- Parties
    payer_id UUID NOT NULL REFERENCES members(id),
    payee_id UUID NOT NULL REFERENCES members(id),
    
    -- Processing Dates
    processed_at TIMESTAMP WITH TIME ZONE,
    settled_at TIMESTAMP WITH TIME ZONE,
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_transactions_partnership_id ON transactions (partnership_id);
CREATE INDEX idx_transactions_status ON transactions (status);
CREATE INDEX idx_transactions_payer_id ON transactions (payer_id);
CREATE INDEX idx_transactions_payee_id ON transactions (payee_id);

-- Commission calculations: Detailed commission breakdown
CREATE TABLE commission_calculations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    transaction_id UUID NOT NULL REFERENCES transactions(id) ON DELETE CASCADE,
    
    base_amount DECIMAL(12,2) NOT NULL,
    commission_rate DECIMAL(5,4) NOT NULL,
    commission_amount DECIMAL(12,2) NOT NULL,
    
    -- Tax and Fees
    tax_amount DECIMAL(12,2) DEFAULT 0,
    processing_fees DECIMAL(12,2) DEFAULT 0,
    net_commission DECIMAL(12,2) NOT NULL,
    
    calculation_date TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_commission_calculations_transaction_id ON commission_calculations (transaction_id);
```

### Creative Collaboration

```sql
-- Collaboration workspaces: Creative collaboration spaces
CREATE TABLE collaboration_workspaces (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    partnership_id UUID NOT NULL REFERENCES partnerships(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    
    -- Settings
    visibility visibility_enum DEFAULT 'partnership',
    settings JSONB DEFAULT '{}',
    
    -- Status
    is_active BOOLEAN DEFAULT TRUE,
    archived_at TIMESTAMP WITH TIME ZONE,
    
    created_by UUID NOT NULL REFERENCES members(id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Workspace members: Access control for collaboration spaces
CREATE TABLE workspace_members (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    workspace_id UUID NOT NULL REFERENCES collaboration_workspaces(id) ON DELETE CASCADE,
    member_id UUID NOT NULL REFERENCES members(id) ON DELETE CASCADE,
    
    role workspace_role_enum DEFAULT 'collaborator',
    permissions workspace_permission_enum[] DEFAULT '{}',
    
    joined_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_active_at TIMESTAMP WITH TIME ZONE,
    
    CONSTRAINT workspace_members_unique UNIQUE (workspace_id, member_id)
);

-- Creative assets: Files and creative materials
CREATE TABLE creative_assets (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    workspace_id UUID NOT NULL REFERENCES collaboration_workspaces(id) ON DELETE CASCADE,
    
    -- File Information
    title VARCHAR(255) NOT NULL,
    description TEXT,
    file_name VARCHAR(255) NOT NULL,
    file_size BIGINT NOT NULL,
    file_type VARCHAR(100) NOT NULL,
    mime_type VARCHAR(100) NOT NULL,
    file_url VARCHAR(1000) NOT NULL,
    
    -- Asset Categorization
    asset_type asset_type_enum NOT NULL,
    tags TEXT[] DEFAULT '{}',
    
    -- Versioning
    version INTEGER DEFAULT 1,
    parent_asset_id UUID REFERENCES creative_assets(id),
    
    -- Approval Workflow
    approval_required BOOLEAN DEFAULT FALSE,
    approval_status approval_status_enum DEFAULT 'pending',
    
    -- Metadata
    uploaded_by UUID NOT NULL REFERENCES members(id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_creative_assets_workspace_id ON creative_assets (workspace_id);
CREATE INDEX idx_creative_assets_asset_type ON creative_assets (asset_type);
CREATE INDEX idx_creative_assets_approval_status ON creative_assets (approval_status);

-- Asset approvals: Approval workflow tracking
CREATE TABLE asset_approvals (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    asset_id UUID NOT NULL REFERENCES creative_assets(id) ON DELETE CASCADE,
    approver_id UUID NOT NULL REFERENCES members(id),
    
    status approval_status_enum DEFAULT 'pending',
    comments TEXT,
    deadline TIMESTAMP WITH TIME ZONE,
    
    approved_at TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    CONSTRAINT asset_approvals_unique UNIQUE (asset_id, approver_id)
);
```

### Analytics & Intelligence

```sql
-- Partnership analytics: Performance tracking
CREATE TABLE partnership_analytics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    partnership_id UUID NOT NULL REFERENCES partnerships(id) ON DELETE CASCADE,
    
    -- Performance Metrics
    completion_percentage DECIMAL(5,2) DEFAULT 0,
    roi_current DECIMAL(8,2),
    roi_projected DECIMAL(8,2),
    performance_score DECIMAL(3,2),
    
    -- Content Metrics
    content_pieces_delivered INTEGER DEFAULT 0,
    total_impressions BIGINT DEFAULT 0,
    total_engagements BIGINT DEFAULT 0,
    engagement_rate DECIMAL(5,4),
    reach BIGINT DEFAULT 0,
    brand_mentions INTEGER DEFAULT 0,
    
    -- Sentiment Analysis
    sentiment_positive DECIMAL(5,4),
    sentiment_neutral DECIMAL(5,4),
    sentiment_negative DECIMAL(5,4),
    overall_sentiment sentiment_enum,
    
    -- Financial Performance
    revenue_generated DECIMAL(12,2),
    cost_per_acquisition DECIMAL(8,2),
    customer_lifetime_value DECIMAL(10,2),
    
    -- Benchmark Comparison
    industry_benchmark_roi DECIMAL(8,2),
    performance_vs_industry DECIMAL(6,3),
    
    -- Temporal Data
    measurement_date DATE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    CONSTRAINT partnership_analytics_unique UNIQUE (partnership_id, measurement_date)
);

CREATE INDEX idx_partnership_analytics_partnership_id ON partnership_analytics (partnership_id);
CREATE INDEX idx_partnership_analytics_measurement_date ON partnership_analytics (measurement_date);

-- Member reputation: Reputation scoring system
CREATE TABLE member_reputation (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    member_id UUID NOT NULL REFERENCES members(id) ON DELETE CASCADE,
    
    -- Reputation Scores
    overall_score DECIMAL(3,2) DEFAULT 3.0,
    partnership_success_rate DECIMAL(5,4) DEFAULT 0,
    communication_score DECIMAL(3,2) DEFAULT 3.0,
    delivery_score DECIMAL(3,2) DEFAULT 3.0,
    collaboration_score DECIMAL(3,2) DEFAULT 3.0,
    
    -- Statistics
    total_partnerships INTEGER DEFAULT 0,
    completed_partnerships INTEGER DEFAULT 0,
    successful_partnerships INTEGER DEFAULT 0,
    average_partnership_value DECIMAL(12,2),
    
    -- Timeline
    member_since TIMESTAMP WITH TIME ZONE,
    last_partnership_date TIMESTAMP WITH TIME ZONE,
    
    -- Calculation Metadata
    last_calculated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    calculation_version INTEGER DEFAULT 1,
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    CONSTRAINT member_reputation_member_id_key UNIQUE (member_id)
);

CREATE INDEX idx_member_reputation_overall_score ON member_reputation (overall_score);
CREATE INDEX idx_member_reputation_partnership_success_rate ON member_reputation (partnership_success_rate);
```

### System Management

```sql
-- Audit logs: System activity tracking
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Entity Information
    entity_type entity_type_enum NOT NULL,
    entity_id UUID NOT NULL,
    action audit_action_enum NOT NULL,
    
    -- Change Tracking
    old_values JSONB,
    new_values JSONB,
    changes JSONB, -- Computed diff
    
    -- Context
    performed_by UUID REFERENCES members(id),
    ip_address INET,
    user_agent TEXT,
    session_id UUID,
    
    -- Additional Context
    metadata JSONB DEFAULT '{}',
    reason TEXT,
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_audit_logs_entity ON audit_logs (entity_type, entity_id);
CREATE INDEX idx_audit_logs_performed_by ON audit_logs (performed_by);
CREATE INDEX idx_audit_logs_created_at ON audit_logs (created_at);

-- Feature flags: Feature toggle management
CREATE TABLE feature_flags (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    flag_name VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    
    -- Status
    is_enabled BOOLEAN DEFAULT FALSE,
    rollout_percentage INTEGER DEFAULT 0, -- 0-100
    
    -- Targeting
    target_conditions JSONB DEFAULT '{}',
    member_whitelist UUID[],
    member_blacklist UUID[],
    
    -- Metadata
    created_by UUID NOT NULL REFERENCES members(id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_feature_flags_flag_name ON feature_flags (flag_name);
CREATE INDEX idx_feature_flags_is_enabled ON feature_flags (is_enabled);

-- Notifications: User notification system
CREATE TABLE notifications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    recipient_id UUID NOT NULL REFERENCES members(id) ON DELETE CASCADE,
    
    -- Notification Content
    type notification_type_enum NOT NULL,
    title VARCHAR(255) NOT NULL,
    message TEXT NOT NULL,
    action_url VARCHAR(1000),
    
    -- Metadata
    data JSONB DEFAULT '{}',
    priority priority_enum DEFAULT 'medium',
    
    -- Status
    is_read BOOLEAN DEFAULT FALSE,
    read_at TIMESTAMP WITH TIME ZONE,
    
    -- Delivery
    delivery_channels notification_channel_enum[] DEFAULT '{in_app}',
    email_sent_at TIMESTAMP WITH TIME ZONE,
    push_sent_at TIMESTAMP WITH TIME ZONE,
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_notifications_recipient_id ON notifications (recipient_id);
CREATE INDEX idx_notifications_is_read ON notifications (is_read);
CREATE INDEX idx_notifications_created_at ON notifications (created_at);
```

---

## Enums and Types

```sql
-- Create custom types and enums
CREATE TYPE subscription_tier_enum AS ENUM ('basic', 'pro', 'enterprise');
CREATE TYPE member_status_enum AS ENUM ('active', 'inactive', 'suspended', 'deleted');
CREATE TYPE company_size_enum AS ENUM ('startup', 'small', 'medium', 'large', 'enterprise');
CREATE TYPE revenue_range_enum AS ENUM ('under_1m', '1m_10m', '10m_100m', '100m_1b', 'over_1b');
CREATE TYPE partnership_type_enum AS ENUM ('content_collaboration', 'brand_sponsorship', 'co_marketing', 'licensing', 'distribution');
CREATE TYPE goal_type_enum AS ENUM ('market_expansion', 'revenue_growth', 'brand_elevation', 'ip_monetization', 'audience_growth');
CREATE TYPE priority_enum AS ENUM ('low', 'medium', 'high', 'critical');
CREATE TYPE goal_status_enum AS ENUM ('active', 'completed', 'paused', 'cancelled');
CREATE TYPE verification_type_enum AS ENUM ('identity', 'business', 'financial', 'social_media');
CREATE TYPE verification_status_enum AS ENUM ('pending', 'approved', 'rejected', 'expired');
CREATE TYPE auth_provider_enum AS ENUM ('google', 'linkedin', 'facebook', 'twitter', 'email');
CREATE TYPE partnership_status_enum AS ENUM ('pending', 'active', 'completed', 'cancelled', 'disputed');
CREATE TYPE partnership_stage_enum AS ENUM ('discovery', 'initial_contact', 'negotiation', 'contract_review', 'execution', 'performance_tracking', 'completion');
CREATE TYPE partnership_role_enum AS ENUM ('brand_sponsor', 'content_creator', 'distributor', 'agency_representative', 'platform_owner');
CREATE TYPE partnership_permission_enum AS ENUM ('view_details', 'edit_content', 'approve_content', 'approve_payments', 'view_analytics', 'manage_members');
CREATE TYPE currency_enum AS ENUM ('USD', 'EUR', 'GBP', 'CAD', 'AUD', 'JPY');
CREATE TYPE payment_structure_enum AS ENUM ('upfront', 'milestone_based', 'performance_based', 'revenue_share', 'hybrid');
CREATE TYPE milestone_status_enum AS ENUM ('pending', 'in_progress', 'completed', 'overdue', 'cancelled');
CREATE TYPE payment_status_enum AS ENUM ('pending', 'paid', 'overdue', 'cancelled');
CREATE TYPE transaction_type_enum AS ENUM ('partnership_payment', 'commission', 'refund', 'subscription_payment', 'bonus');
CREATE TYPE transaction_status_enum AS ENUM ('pending', 'processing', 'completed', 'failed', 'cancelled', 'refunded');
CREATE TYPE payment_method_enum AS ENUM ('stripe', 'paypal', 'wire_transfer', 'ach', 'credit_card');
CREATE TYPE commission_status_enum AS ENUM ('pending', 'calculated', 'paid', 'disputed');
CREATE TYPE visibility_enum AS ENUM ('public', 'network', 'partnership', 'private');
CREATE TYPE workspace_role_enum AS ENUM ('owner', 'admin', 'collaborator', 'viewer');
CREATE TYPE workspace_permission_enum AS ENUM ('view', 'comment', 'upload', 'edit', 'delete', 'approve', 'manage_members');
CREATE TYPE asset_type_enum AS ENUM ('image', 'video', 'audio', 'document', 'presentation', 'design_file', 'other');
CREATE TYPE approval_status_enum AS ENUM ('pending', 'approved', 'rejected', 'revision_requested');
CREATE TYPE sentiment_enum AS ENUM ('positive', 'neutral', 'negative');
CREATE TYPE entity_type_enum AS ENUM ('member', 'partnership', 'milestone', 'transaction', 'workspace', 'asset');
CREATE TYPE audit_action_enum AS ENUM ('created', 'updated', 'deleted', 'approved', 'rejected', 'completed');
CREATE TYPE notification_type_enum AS ENUM ('partnership_created', 'milestone_completed', 'payment_received', 'approval_required', 'system_update');
CREATE TYPE notification_channel_enum AS ENUM ('in_app', 'email', 'sms', 'push');
```

---

## Neo4j Graph Schema

### Node Types and Relationships

```cypher
// Member nodes
CREATE (m:Member {
    id: 'mem_123456789',
    name: 'Sarah Bennett',
    type: 'brand_professional',
    company: 'Global Brands Inc',
    industry: 'consumer_goods',
    subscription_tier: 'enterprise'
})

// Brand nodes
CREATE (b:Brand {
    id: 'brand_001',
    name: 'EcoSustain Products',
    industry: 'sustainability',
    market_cap: 'large',
    target_demographics: ['millennials', 'gen_z'],
    brand_values: ['sustainability', 'innovation', 'authenticity']
})

// Content nodes
CREATE (c:Content {
    id: 'content_001',
    title: 'Sustainable Living Documentary Series',
    type: 'video_series',
    genre: 'documentary',
    audience_size: 2500000,
    primary_demographics: ['18-34', 'environmentally_conscious']
})

// Trend nodes
CREATE (t:Trend {
    id: 'trend_001',
    name: 'sustainable_consumption',
    category: 'lifestyle',
    momentum_score: 0.89,
    growth_rate: 0.23,
    relevance_keywords: ['eco-friendly', 'sustainable', 'ethical']
})

// Geographic nodes
CREATE (g:Geographic {
    id: 'geo_001',
    name: 'North America',
    type: 'region',
    market_size: 'large',
    regulatory_environment: 'mature'
})

// Demographic nodes
CREATE (d:Demographic {
    id: 'demo_001',
    segment: 'millennials',
    age_range: '25-40',
    characteristics: ['tech_savvy', 'value_conscious', 'brand_loyal'],
    size: 72000000
})
```

### Key Relationships

```cypher
// Affinity relationships
(:Member)-[:HAS_AFFINITY_WITH {strength: 0.85, last_calculated: datetime()}]->(:Trend)
(:Brand)-[:ALIGNS_WITH {alignment_score: 0.92}]->(:Trend)
(:Content)-[:APPEALS_TO {engagement_rate: 0.087}]->(:Demographic)

// Business relationships
(:Member)-[:WORKS_FOR]->(:Brand)
(:Member)-[:CREATES]->(:Content)
(:Brand)-[:SPONSORS {amount: 250000, duration: '6 months'}]->(:Content)
(:Brand)-[:TARGETS]->(:Demographic)
(:Brand)-[:OPERATES_IN]->(:Geographic)

// Partnership relationships
(:Member)-[:PARTNERS_WITH {
    partnership_id: 'part_001',
    status: 'active',
    success_score: 4.3,
    total_value: 250000,
    start_date: date('2024-09-01'),
    end_date: date('2024-12-31')
}]->(:Member)

// Cultural relationships
(:Trend)-[:INFLUENCES]->(:Demographic)
(:Trend)-[:EMERGES_IN]->(:Geographic)
(:Content)-[:REFLECTS]->(:Trend)
(:Brand)-[:CAPITALIZES_ON]->(:Trend)

// Similarity relationships
(:Member)-[:SIMILAR_TO {similarity_score: 0.78, basis: 'industry_and_values'}]->(:Member)
(:Brand)-[:COMPETES_WITH {market_overlap: 0.65}]->(:Brand)
(:Content)-[:SIMILAR_CONTENT {content_similarity: 0.84}]->(:Content)
```

### Graph Queries for Affinity Matching

```cypher
-- Find potential partners based on shared affinities
MATCH (m1:Member {id: $member_id})-[:HAS_AFFINITY_WITH]-(t:Trend)-[:HAS_AFFINITY_WITH]-(m2:Member)
WHERE m1 <> m2
WITH m2, collect(t) as shared_trends, count(t) as affinity_count
WHERE affinity_count >= 3
RETURN m2, shared_trends, affinity_count
ORDER BY affinity_count DESC
LIMIT 20

-- Discover partnership opportunities through cultural trends
MATCH (m:Member {id: $member_id})-[:HAS_AFFINITY_WITH]-(trend:Trend)
WHERE trend.momentum_score > 0.7
MATCH (trend)-[:INFLUENCES]-(demo:Demographic)-[:TARGETED_BY]-(brand:Brand)
WHERE NOT (m)-[:PARTNERS_WITH]-(brand)
RETURN brand, trend, demo, trend.momentum_score as opportunity_score
ORDER BY opportunity_score DESC

-- Find successful partnership patterns
MATCH (m1:Member)-[p:PARTNERS_WITH {status: 'completed'}]-(m2:Member)
WHERE p.success_score > 4.0
MATCH (m1)-[:HAS_AFFINITY_WITH]-(t:Trend)-[:HAS_AFFINITY_WITH]-(m2)
RETURN t, avg(p.success_score) as avg_success, count(p) as partnership_count
ORDER BY avg_success DESC, partnership_count DESC
```

---

## Redis Caching Strategy

### Cache Key Patterns

```redis
# Member data caching
member:profile:{member_id}           # TTL: 1800 seconds
member:goals:{member_id}             # TTL: 3600 seconds
member:reputation:{member_id}        # TTL: 7200 seconds

# Affinity Engine caching
affinity:recommendations:{member_id} # TTL: 3600 seconds
affinity:graph:{member_id}          # TTL: 1800 seconds
cultural:trends:global              # TTL: 900 seconds

# Partnership data
partnership:{partnership_id}         # TTL: 300 seconds
partnership:analytics:{partnership_id} # TTL: 1800 seconds

# Session management
session:{session_token}             # TTL: 86400 seconds
user:permissions:{member_id}        # TTL: 3600 seconds

# Rate limiting
ratelimit:{member_id}:{endpoint}    # TTL: 3600 seconds
ratelimit:global:{ip}               # TTL: 3600 seconds

# Real-time features
notifications:{member_id}           # TTL: 86400 seconds
workspace:activity:{workspace_id}   # TTL: 1800 seconds
```

### Redis Data Structures

```redis
# Sorted sets for rankings and recommendations
ZADD affinity:scores:{member_id} 0.94 "mem_partner_001" 0.87 "mem_partner_002"

# Hash sets for complex objects
HMSET partnership:part_001 
    "title" "Sustainable Living Partnership"
    "status" "active" 
    "stage" "execution"
    "total_value" "250000"
    "commission_rate" "0.08"

# Lists for activity feeds and notifications
LPUSH activity:feed:{member_id} "partnership:part_001:milestone_completed"

# Sets for relationships and permissions
SADD member:permissions:{member_id} "view_analytics" "approve_content"
```

---

## Pinecone Vector Database

### Vector Embedding Strategy

```python
# Member profile embeddings
member_embeddings = {
    'profile_text': encode_profile_text(member.bio, member.company_description),
    'goals_vector': encode_goals(member.goals),
    'industry_vector': encode_industry(member.industry, member.specializations),
    'values_vector': encode_values(member.brand_values, member.partnership_preferences)
}

# Content embeddings
content_embeddings = {
    'content_description': encode_content(content.title, content.description),
    'audience_vector': encode_audience(content.target_demographics),
    'genre_vector': encode_genre(content.genre, content.style),
    'performance_vector': encode_performance(content.engagement_metrics)
}

# Cultural trend embeddings
trend_embeddings = {
    'trend_description': encode_trend(trend.name, trend.description),
    'momentum_vector': encode_momentum(trend.momentum_score, trend.growth_rate),
    'demographic_appeal': encode_demographics(trend.target_demographics)
}
```

### Index Configuration

```python
import pinecone

# Initialize Pinecone
pinecone.init(api_key="your-api-key", environment="production")

# Create indexes for different entity types
indexes = {
    'member-profiles': {
        'dimension': 1536,  # OpenAI embedding dimension
        'metric': 'cosine',
        'pods': 1,
        'pod_type': 'p1.x1'
    },
    'content-library': {
        'dimension': 1536,
        'metric': 'cosine',
        'pods': 2,
        'pod_type': 'p1.x1'
    },
    'cultural-trends': {
        'dimension': 1536,
        'metric': 'cosine',
        'pods': 1,
        'pod_type': 'p1.x1'
    }
}

# Create index
index = pinecone.Index("member-profiles")

# Upsert member embeddings
index.upsert([
    ("mem_123456789", member_embedding, {
        "member_id": "mem_123456789",
        "subscription_tier": "enterprise",
        "industry": "consumer_goods",
        "last_active": "2024-08-20T14:30:00Z"
    })
])

# Query for similar members
results = index.query(
    vector=query_member_embedding,
    top_k=50,
    filter={
        "industry": {"$in": ["consumer_goods", "marketing", "media"]},
        "subscription_tier": {"$in": ["pro", "enterprise"]}
    },
    include_metadata=True
)
```

---

## Database Backup and Recovery

### PostgreSQL Backup Strategy

```bash
#!/bin/bash
# Automated backup script
pg_dump -h $DB_HOST -U $DB_USER -d teraffi_production \
    --format=custom \
    --compress=9 \
    --verbose \
    --file="/backups/teraffi_$(date +%Y%m%d_%H%M%S).dump"

# Point-in-time recovery setup
postgres -c wal_level=replica \
         -c max_wal_senders=10 \
         -c archive_mode=on \
         -c archive_command='cp %p /backups/wal/%f'
```

### Neo4j Backup Strategy

```bash
# Online backup for Neo4j
neo4j-admin backup --from=bolt://localhost:7687 \
    --backup-dir=/backups/neo4j \
    --name=graph-$(date +%Y%m%d)
```

This comprehensive database schema provides the foundation for TERAFFI's multi-database architecture, supporting the Affinity Engine™, partnership management, creative collaboration, and analytics while maintaining data integrity, performance, and scalability.