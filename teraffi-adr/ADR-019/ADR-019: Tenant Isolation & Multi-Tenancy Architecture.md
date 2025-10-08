# ADR-019: Tenant Isolation & Multi-Tenancy Architecture

**Status:** Proposed  
**Date:** 2025-10-01  
**Decision Makers:** Platform Architecture Team  
**Related ADRs:** ADR-001 (Runtime), ADR-007 (PostgreSQL), ADR-011 (Neo4j), ADR-009 (Redis)

---

## Context

TERAFFI operates as a multi-tenant SaaS platform where multiple organizations (brands, agencies, IP owners) share the same infrastructure while maintaining complete data isolation. Each tenant must have:

**Data Isolation:**
- No tenant can access another tenant's data
- Even database administrators should have limited cross-tenant visibility
- Breaches must be contained to single tenant

**Performance Isolation:**
- One tenant's heavy usage cannot degrade others' experience
- Resource quotas per tenant tier
- Rate limiting per tenant

**Customization:**
- Tenant-specific branding and configuration
- Custom workflows per tenant
- Feature flags per tenant

**Compliance:**
- SOC 2 Type II requirements
- GDPR data residency and deletion
- Audit trails per tenant

### Current Gap

Without proper multi-tenancy:
- **Security Risk**: Inadequate isolation = data breach exposure
- **Performance Issues**: No resource limits = noisy neighbor problem
- **Compliance Gaps**: Cannot meet SOC 2/GDPR requirements
- **Scale Limitations**: Cannot onboard enterprise customers

### Requirements

**Functional:**
1. Tenant identification and context propagation
2. Row-level security in all databases
3. Tenant-scoped queries across all services
4. Tenant-specific configuration storage
5. Tenant provisioning/deprovisioning
6. Tenant usage tracking and quotas

**Non-Functional:**
1. Zero cross-tenant data leakage
2. <5% performance overhead for isolation
3. Support 1000+ tenants per database
4. Tenant data deletable within 30 days (GDPR)
5. SOC 2 compliant audit logging

---

## Decision

**Implement shared database, shared schema multi-tenancy** with:
1. **tenant_id column** in all tables for row-level filtering
2. **Context propagation** through entire request lifecycle
3. **Database-level security** with Row-Level Security (RLS) policies
4. **Application-level enforcement** as defense-in-depth
5. **Tenant registry** for metadata and configuration
6. **Resource quotas** enforced at API gateway and service layers

**Architecture:**
```
API Gateway
    ↓
[Tenant Context Middleware] ← Extract from JWT
    ↓
Business Logic Services
    ↓ (tenant_id in all queries)
Databases (PostgreSQL/Neo4j/Redis)
    ↓
[RLS Policies / Query Filters]
    ↓
Tenant-Scoped Data Only
```

---

## Rationale

### Multi-Tenancy Patterns Comparison

**Pattern 1: Separate Database per Tenant**
- **Pros**: Perfect isolation, simple queries, easy backup/restore per tenant
- **Cons**: Expensive, operationally complex (1000s of databases), schema changes difficult
- **Decision**: Too expensive and operationally complex for TERAFFI's scale

**Pattern 2: Separate Schema per Tenant**
- **Pros**: Good isolation, moderate cost
- **Cons**: 1000s of schemas hard to manage, connection pooling complex
- **Decision**: Better than separate DBs but still operationally heavy

**Pattern 3: Shared Database, Shared Schema (with tenant_id)**
- **Pros**: Cost-effective, operationally simple, scales to 1000s of tenants
- **Cons**: Requires discipline to always filter by tenant_id
- **Decision**: Best balance for TERAFFI's B2B SaaS model

### Why Row-Level Security (RLS)

**Defense in Depth:**
- Application filters tenant_id (primary defense)
- Database RLS policies enforce at DB level (backup defense)
- Even if developer forgets WHERE tenant_id = X, RLS catches it

**Compliance:**
- SOC 2 auditors want database-level controls
- GDPR requires demonstrable isolation
- Audit logs show enforcement at multiple layers

---

## Implementation Details

### 1. Tenant Context Propagation

```typescript
// packages/core/src/middleware/tenant-context.ts

export interface TenantContext {
  tenant_id: string;
  tenant_name: string;
  tier: 'free' | 'professional' | 'enterprise';
  features: string[];
  quotas: {
    api_requests_per_hour: number;
    partnerships_per_month: number;
    storage_gb: number;
  };
}

export class TenantContextMiddleware {
  async handle(request: FastifyRequest, reply: FastifyReply): Promise<void> {
    // Extract tenant from JWT
    const token = request.headers.authorization?.replace('Bearer ', '');
    if (!token) {
      throw new UnauthorizedError('Missing authentication token');
    }

    const decoded = await this.jwtService.verify(token);
    
    // Load tenant context
    const tenant = await this.tenantRegistry.get(decoded.tenant_id);
    if (!tenant) {
      throw new UnauthorizedError('Invalid tenant');
    }

    if (!tenant.active) {
      throw new ForbiddenError('Tenant account suspended');
    }

    // Store in request context
    request.tenantContext = {
      tenant_id: tenant.id,
      tenant_name: tenant.name,
      tier: tenant.tier,
      features: tenant.features,
      quotas: tenant.quotas
    };

    // Set in async local storage for downstream services
    await this.asyncLocalStorage.run(request.tenantContext, async () => {
      // Continue request processing
    });
  }
}
```

### 2. PostgreSQL Row-Level Security

```sql
-- Enable RLS on all tenant-scoped tables

-- Example: partnerships table
ALTER TABLE partnerships ENABLE ROW LEVEL SECURITY;

-- Create policy that only allows access to rows matching current tenant
CREATE POLICY tenant_isolation_policy ON partnerships
  FOR ALL
  USING (tenant_id = current_setting('app.current_tenant_id')::UUID);

-- Similarly for all tables
ALTER TABLE members ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation_policy ON members
  FOR ALL
  USING (tenant_id = current_setting('app.current_tenant_id')::UUID);

ALTER TABLE brands ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation_policy ON brands
  FOR ALL
  USING (tenant_id = current_setting('app.current_tenant_id')::UUID);

-- Set tenant context at connection level
-- Called at start of every database transaction
SET LOCAL app.current_tenant_id = 'tenant-uuid-here';
```

```typescript
// packages/database/src/tenant-scoped-db.ts

export class TenantScopedDatabase {
  constructor(private pool: Pool) {}

  async withTenant<T>(tenantId: string, callback: (db: Kysely<Database>) => Promise<T>): Promise<T> {
    return await this.pool.connect(async (connection) => {
      // Set tenant context for RLS
      await connection.query(`SET LOCAL app.current_tenant_id = '${tenantId}'`);
      
      // Create Kysely instance with this connection
      const db = new Kysely<Database>({
        dialect: new PostgresDialect({ pool: connection })
      });

      // Execute callback with tenant-scoped db
      return await callback(db);
    });
  }

  // Convenience wrapper that gets tenant from context
  async execute<T>(callback: (db: Kysely<Database>) => Promise<T>): Promise<T> {
    const context = this.getTenantContext();
    if (!context) {
      throw new Error('Tenant context not set');
    }
    
    return await this.withTenant(context.tenant_id, callback);
  }

  private getTenantContext(): TenantContext | null {
    // Get from async local storage
    return asyncLocalStorage.getStore() as TenantContext;
  }
}
```

### 3. Neo4j Tenant Isolation

```typescript
// packages/graph/src/tenant-scoped-neo4j.ts

export class TenantScopedNeo4j {
  constructor(private driver: Neo4jDriver) {}

  session(tenantId: string): Session {
    return this.driver.session({
      defaultAccessMode: neo4j.session.READ,
      database: 'neo4j'
    });
  }

  // Wrapper that adds tenant_id to all queries
  async run(
    tenantId: string,
    query: string,
    parameters: Record<string, any> = {}
  ): Promise<Result> {
    const session = this.session(tenantId);
    
    try {
      // Inject tenant filter into query
      const tenantScopedQuery = this.addTenantFilter(query);
      
      return await session.run(tenantScopedQuery, {
        ...parameters,
        tenant_id: tenantId
      });
    } finally {
      await session.close();
    }
  }

  private addTenantFilter(query: string): string {
    // Parse and inject WHERE tenant_id = $tenant_id
    // This is simplified - production would use proper query parser
    
    if (query.includes('MATCH') && !query.includes('tenant_id')) {
      // Add tenant filter to all MATCH clauses
      return query.replace(
        /MATCH \((\w+)(?::(\w+))?\)/g,
        'MATCH ($1$2 {tenant_id: $tenant_id})'
      );
    }
    
    return query;
  }
}
```

### 4. Redis Tenant Scoping

```typescript
// packages/cache/src/tenant-scoped-redis.ts

export class TenantScopedRedis {
  constructor(private client: RedisClient) {}

  // All keys prefixed with tenant_id
  private scopeKey(tenantId: string, key: string): string {
    return `tenant:${tenantId}:${key}`;
  }

  async get(tenantId: string, key: string): Promise<string | null> {
    return await this.client.get(this.scopeKey(tenantId, key));
  }

  async set(
    tenantId: string,
    key: string,
    value: string,
    ttl?: number
  ): Promise<void> {
    const scopedKey = this.scopeKey(tenantId, key);
    
    if (ttl) {
      await this.client.setex(scopedKey, ttl, value);
    } else {
      await this.client.set(scopedKey, value);
    }
  }

  async delete(tenantId: string, key: string): Promise<void> {
    await this.client.del(this.scopeKey(tenantId, key));
  }

  // Delete all keys for a tenant (GDPR right to deletion)
  async deleteAllForTenant(tenantId: string): Promise<void> {
    const pattern = `tenant:${tenantId}:*`;
    const keys = await this.client.keys(pattern);
    
    if (keys.length > 0) {
      await this.client.del(...keys);
    }
  }
}
```

### 5. Tenant Registry

```typescript
// packages/tenant-management/src/tenant-registry.ts

interface Tenant {
  id: string;
  name: string;
  slug: string;  // URL-friendly identifier
  tier: 'free' | 'professional' | 'enterprise';
  features: string[];
  quotas: TenantQuotas;
  config: TenantConfig;
  active: boolean;
  created_at: Date;
  updated_at: Date;
}

interface TenantQuotas {
  api_requests_per_hour: number;
  partnerships_per_month: number;
  storage_gb: number;
  team_members: number;
}

interface TenantConfig {
  branding: {
    logo_url?: string;
    primary_color?: string;
    custom_domain?: string;
  };
  features: {
    advanced_analytics: boolean;
    white_label: boolean;
    api_access: boolean;
    sso: boolean;
  };
  integrations: {
    docusign_enabled: boolean;
    slack_enabled: boolean;
  };
}

export class TenantRegistry {
  constructor(private db: Database, private cache: RedisClient) {}

  async get(tenantId: string): Promise<Tenant | null> {
    // Check cache first
    const cacheKey = `tenant:${tenantId}`;
    const cached = await this.cache.get(cacheKey);
    
    if (cached) {
      return JSON.parse(cached);
    }

    // Load from database
    const tenant = await this.db
      .selectFrom('tenants')
      .selectAll()
      .where('id', '=', tenantId)
      .executeTakeFirst();

    if (tenant) {
      // Cache for 5 minutes
      await this.cache.setex(cacheKey, 300, JSON.stringify(tenant));
    }

    return tenant || null;
  }

  async create(data: Omit<Tenant, 'id' | 'created_at' | 'updated_at'>): Promise<Tenant> {
    const tenant: Tenant = {
      id: generateId('tenant_'),
      ...data,
      created_at: new Date(),
      updated_at: new Date()
    };

    await this.db.insertInto('tenants').values(tenant).execute();

    // Invalidate cache
    await this.cache.del(`tenant:${tenant.id}`);

    return tenant;
  }

  async update(tenantId: string, data: Partial<Tenant>): Promise<void> {
    await this.db
      .updateTable('tenants')
      .set({ ...data, updated_at: new Date() })
      .where('id', '=', tenantId)
      .execute();

    // Invalidate cache
    await this.cache.del(`tenant:${tenantId}`);
  }

  async deactivate(tenantId: string): Promise<void> {
    await this.update(tenantId, { active: false });
  }
}
```

### 6. Resource Quotas & Rate Limiting

```typescript
// packages/core/src/middleware/quota-enforcement.ts

export class QuotaEnforcementMiddleware {
  async handle(request: FastifyRequest, reply: FastifyReply): Promise<void> {
    const context = request.tenantContext;
    if (!context) return;

    // Check API rate limit
    const apiRateLimitKey = `quota:api:${context.tenant_id}`;
    const currentCount = await this.redis.incr(apiRateLimitKey);
    
    if (currentCount === 1) {
      // Set expiry on first request in window
      await this.redis.expire(apiRateLimitKey, 3600);  // 1 hour window
    }

    if (currentCount > context.quotas.api_requests_per_hour) {
      throw new RateLimitError(
        `API rate limit exceeded: ${context.quotas.api_requests_per_hour} requests/hour`
      );
    }

    // Add rate limit headers
    reply.header('X-RateLimit-Limit', context.quotas.api_requests_per_hour);
    reply.header('X-RateLimit-Remaining', Math.max(0, context.quotas.api_requests_per_hour - currentCount));
    reply.header('X-RateLimit-Reset', await this.redis.ttl(apiRateLimitKey));
  }

  async checkPartnershipQuota(tenantId: string): Promise<boolean> {
    const context = await this.tenantRegistry.get(tenantId);
    
    // Count partnerships this month
    const startOfMonth = new Date();
    startOfMonth.setDate(1);
    startOfMonth.setHours(0, 0, 0, 0);

    const count = await this.db
      .selectFrom('partnerships')
      .select(db.fn.count<number>('id').as('count'))
      .where('tenant_id', '=', tenantId)
      .where('created_at', '>=', startOfMonth)
      .executeTakeFirstOrThrow();

    return count.count < context.quotas.partnerships_per_month;
  }
}
```

### 7. Tenant Provisioning

```typescript
// packages/tenant-management/src/provisioning.ts

export class TenantProvisioningService {
  async provision(data: {
    name: string;
    admin_email: string;
    tier: 'free' | 'professional' | 'enterprise';
  }): Promise<Tenant> {
    // 1. Create tenant record
    const tenant = await this.tenantRegistry.create({
      name: data.name,
      slug: slugify(data.name),
      tier: data.tier,
      features: this.getFeaturesForTier(data.tier),
      quotas: this.getQuotasForTier(data.tier),
      config: this.getDefaultConfig(),
      active: true
    });

    // 2. Create admin user
    await this.userService.create({
      email: data.admin_email,
      tenant_id: tenant.id,
      role: 'admin'
    });

    // 3. Set up default data (optional)
    await this.seedDefaultData(tenant.id);

    // 4. Send welcome email
    await this.emailService.sendWelcome({
      to: data.admin_email,
      tenant_name: tenant.name
    });

    // 5. Log provisioning event
    await this.auditLog.record({
      event: 'tenant_provisioned',
      tenant_id: tenant.id,
      metadata: { tier: data.tier }
    });

    return tenant;
  }

  async deprovision(tenantId: string): Promise<void> {
    // 1. Deactivate tenant
    await this.tenantRegistry.deactivate(tenantId);

    // 2. Schedule data deletion (30 day grace period for GDPR)
    await this.scheduleDataDeletion(tenantId, addDays(new Date(), 30));

    // 3. Notify users
    const users = await this.userService.listByTenant(tenantId);
    for (const user of users) {
      await this.emailService.send({
        to: user.email,
        subject: 'Account Deactivated',
        template: 'tenant_deactivated'
      });
    }
  }

  async deleteData(tenantId: string): Promise<void> {
    // GDPR right to deletion - permanently delete all tenant data

    await this.db.transaction(async (trx) => {
      // Delete from all tables
      await trx.deleteFrom('partnerships').where('tenant_id', '=', tenantId).execute();
      await trx.deleteFrom('members').where('tenant_id', '=', tenantId).execute();
      await trx.deleteFrom('brands').where('tenant_id', '=', tenantId).execute();
      await trx.deleteFrom('deal_documents').where('tenant_id', '=', tenantId).execute();
      // ... all other tables

      // Delete from Neo4j
      await this.neo4j.run(`
        MATCH (n {tenant_id: $tenantId})
        DETACH DELETE n
      `, { tenantId });

      // Delete from Redis
      await this.redis.deleteAllForTenant(tenantId);

      // Delete from blob storage
      await this.blobStorage.deleteContainer(`tenant-${tenantId}`);

      // Finally, delete tenant record
      await trx.deleteFrom('tenants').where('id', '=', tenantId).execute();
    });

    await this.auditLog.record({
      event: 'tenant_data_deleted',
      tenant_id: tenantId,
      timestamp: new Date()
    });
  }

  private getFeaturesForTier(tier: string): string[] {
    const features = {
      free: ['basic_matching', 'limited_analytics'],
      professional: ['basic_matching', 'advanced_analytics', 'api_access'],
      enterprise: ['basic_matching', 'advanced_analytics', 'api_access', 'white_label', 'sso', 'priority_support']
    };
    return features[tier] || features.free;
  }

  private getQuotasForTier(tier: string): TenantQuotas {
    const quotas = {
      free: {
        api_requests_per_hour: 100,
        partnerships_per_month: 5,
        storage_gb: 1,
        team_members: 3
      },
      professional: {
        api_requests_per_hour: 1000,
        partnerships_per_month: 50,
        storage_gb: 10,
        team_members: 10
      },
      enterprise: {
        api_requests_per_hour: 10000,
        partnerships_per_month: 999999,
        storage_gb: 100,
        team_members: 999999
      }
    };
    return quotas[tier] || quotas.free;
  }
}
```

---

## Database Schema

```sql
-- Tenants table
CREATE TABLE tenants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(100) UNIQUE NOT NULL,
  tier VARCHAR(50) NOT NULL,
  features TEXT[],
  quotas JSONB NOT NULL,
  config JSONB,
  active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_tenants_slug ON tenants(slug);
CREATE INDEX idx_tenants_active ON tenants(active);

-- Add tenant_id to all existing tables
ALTER TABLE members ADD COLUMN tenant_id UUID REFERENCES tenants(id);
ALTER TABLE brands ADD COLUMN tenant_id UUID REFERENCES tenants(id);
ALTER TABLE partnerships ADD COLUMN tenant_id UUID REFERENCES tenants(id);
ALTER TABLE deal_documents ADD COLUMN tenant_id UUID REFERENCES tenants(id);
-- ... all other tables

-- Create indexes on tenant_id for performance
CREATE INDEX idx_members_tenant ON members(tenant_id);
CREATE INDEX idx_brands_tenant ON brands(tenant_id);
CREATE INDEX idx_partnerships_tenant ON partnerships(tenant_id);
-- ... all other tables

-- Tenant audit log
CREATE TABLE tenant_audit_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID REFERENCES tenants(id),
  event VARCHAR(100),
  actor_id UUID,
  metadata JSONB,
  timestamp TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_tenant_audit_log_tenant ON tenant_audit_log(tenant_id, timestamp DESC);
```

---

## Migration Path

### Phase 1: Add Tenant Infrastructure (Months 1-2)
- Add tenants table
- Add tenant_id columns to all tables
- Tenant registry service
- Context middleware

### Phase 2: Enforcement (Months 3-4)
- PostgreSQL RLS policies
- Neo4j tenant filters
- Redis key scoping
- Application-level enforcement

### Phase 3: Quotas & Limits (Months 5-6)
- Rate limiting
- Resource quotas
- Usage tracking
- Billing integration

### Phase 4: Operations (Months 7-8)
- Provisioning/deprovisioning
- Data deletion (GDPR)
- Tenant monitoring
- SOC 2 compliance

---

## Success Metrics

**Security:**
- Zero cross-tenant data leakage incidents
- 100% of queries include tenant filtering
- Pass SOC 2 Type II audit

**Performance:**
- <5% overhead from isolation
- <100ms added latency from context propagation
- Support 1000+ active tenants

**Operations:**
- <5 minutes to provision new tenant
- <30 days to complete data deletion
- 99.9% uptime per tenant

---

## References

- [Multi-Tenancy Patterns](https://docs.microsoft.com/en-us/azure/architecture/patterns/category/multitenant)
- [PostgreSQL Row-Level Security](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)
- [SOC 2 Compliance Guide](https://www.aicpa.org/resources/landing/soc-2-compliance)
- [GDPR Right to Deletion](https://gdpr-info.eu/art-17-gdpr/)

---
