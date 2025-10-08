# TERAFFI Platform - Complete Migration & Deployment Plan

## Overview
This document outlines the complete migration strategy to split teraffi-demo-app into 4 production repositories and deploy ALL services to Azure with Kong Gateway on teraffi.io domain.

## Repository Structure

### 1. teraffi-platform (Core Business Logic)
**Purpose**: Core platform services, business logic, and APIs

**Services to Migrate**:
- Authentication Service (from `/backend/src/controllers/AuthController.ts`)
- User Management Service (from `/backend/src/controllers/UserController.ts`)
- Partnership Service (from `/backend/src/controllers/PartnershipController.ts`)
- Affinity Service (from `/backend/src/controllers/AffinityController.ts`)
- Dashboard Service (from `/backend/src/controllers/DashboardController.ts`)
- Search Service (from `/backend/src/controllers/SearchController.ts`)
- Member Profile Service (from `/backend/src/controllers/MemberProfileController.ts`)

**Database Components**:
- PostgreSQL main database schemas
- Multi-tenant architecture tables
- Business entity models

**Key Features to Implement**:
- Multi-tenant hierarchy (accounts → organizations → users)
- Two-tier admin system (platform admins vs account/org admins)
- Row-level security for tenant isolation
- JWT authentication with tenant context

### 2. teraffi-ai (AI/ML Services)
**Purpose**: All AI, ML, and GraphRAG services

**Services to Migrate**:
- GraphRAG Service (from `/backend/src/services/graphrag/`)
- LLM Gateway (from `/backend/src/services/llm-gateway/`)
- Entity Resolution (from `/backend/src/services/entity-resolution/`)
- Data Crawler (from `/backend/src/services/data-crawler/`)
- Embedding Service (from `/backend/src/services/embeddings/`)
- Triple Extractor (from `/backend/src/ai/triple-extractor.ts`)
- Affinity Engine (from `/backend/src/ai/affinity-engine.ts`)
- Cultural Intelligence (from `/backend/src/ai/cultural-intelligence.ts`)

**Infrastructure Components**:
- Neo4j with DozerDB
- PostgreSQL with pgvector
- Ollama for local LLM
- Redis for GraphRAG (port 6380)

### 3. teraffi-infrastructure (DevOps & Infrastructure)
**Purpose**: All infrastructure, deployment, and DevOps code

**Components to Migrate**:
- Kong Gateway configuration
- Docker Compose files
- Azure Bicep/ARM templates
- CI/CD pipelines (GitHub Actions)
- Monitoring configurations (Prometheus, Grafana)
- Nginx configurations
- Environment configurations

**New Components to Create**:
- Kong declarative configuration
- Azure Container Apps definitions
- Terraform/Bicep for Azure resources
- SSL certificate management for teraffi.io

### 4. teraffi-frontend (UI Application)
**Purpose**: Next.js frontend application

**Components to Migrate**:
- All `/src/app/` pages and layouts
- All `/src/components/` React components
- Design system components
- Authentication flows
- Dashboard interfaces

**Enhancements Required**:
- Single app with role-based routing
- Platform admin interface
- Account/org admin interface
- Multi-tenant context switching
- Real-time collaboration features

## Kong API Gateway Configuration

### Service Routes
```yaml
services:
  # Platform Services
  - name: auth-service
    url: http://teraffi-platform-auth:3001
    routes:
      - name: auth-route
        paths: ["/api/v1/auth"]
    plugins:
      - name: cors
      - name: rate-limiting
        config:
          minute: 100

  - name: partnership-service  
    url: http://teraffi-platform-partnerships:3002
    routes:
      - name: partnership-route
        paths: ["/api/v1/partnerships"]
    plugins:
      - name: jwt
      - name: acl
        config:
          whitelist: ["admin", "manager", "analyst"]

  # AI Services
  - name: graphrag-service
    url: http://teraffi-ai-graphrag:4001
    routes:
      - name: graphrag-route
        paths: ["/api/v1/graphrag"]
    plugins:
      - name: jwt
      - name: request-transformer
        config:
          add:
            headers:
              - X-Tenant-ID:$(jwt.account_id)

  - name: llm-gateway
    url: http://teraffi-ai-llm:4002
    routes:
      - name: llm-route
        paths: ["/api/v1/llm"]
    plugins:
      - name: jwt
      - name: rate-limiting
        config:
          minute: 50
```

## Azure Deployment Architecture

### Resource Allocation (Budget: $140-250/month)

#### Container Apps ($120/month)
```yaml
Platform Services (4 containers): $48
  - auth-service: 0.25 vCPU, 0.5GB RAM
  - user-service: 0.25 vCPU, 0.5GB RAM  
  - partnership-service: 0.5 vCPU, 1GB RAM
  - dashboard-service: 0.5 vCPU, 1GB RAM

AI Services (5 containers): $60
  - graphrag-service: 0.5 vCPU, 2GB RAM
  - llm-gateway: 0.25 vCPU, 0.5GB RAM
  - entity-resolution: 0.5 vCPU, 1GB RAM
  - data-crawler: 0.5 vCPU, 1GB RAM
  - embedding-service: 0.5 vCPU, 1GB RAM

Infrastructure (2 containers): $12
  - kong-gateway: 0.5 vCPU, 1GB RAM
  - scheduler: 0.25 vCPU, 0.5GB RAM
```

#### Databases ($80/month)
```yaml
PostgreSQL Flexible Server: $40
  - 2 vCores, 8GB storage
  - Includes pgvector extension
  
Neo4j Community (Container): $20
  - 1 vCPU, 2GB RAM
  - With DozerDB for security
  
Redis Cache: $20
  - 1GB Basic tier
  - For sessions and GraphRAG
```

#### Storage & Networking ($50/month)
```yaml
Application Gateway: $25
  - SSL termination for teraffi.io
  - WAF protection
  
Container Registry: $5
Storage Account: $10
Log Analytics: $10
```

## Database Migration Scripts

### Multi-Tenant Schema
```sql
-- accounts table
CREATE TABLE accounts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    subdomain VARCHAR(100) UNIQUE NOT NULL,
    plan_type VARCHAR(50) DEFAULT 'enterprise',
    status VARCHAR(50) DEFAULT 'active',
    settings JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- organizations table  
CREATE TABLE organizations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID REFERENCES accounts(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    settings JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Update users table
ALTER TABLE users 
ADD COLUMN account_id UUID REFERENCES accounts(id),
ADD COLUMN organization_id UUID REFERENCES organizations(id),
ADD COLUMN is_platform_admin BOOLEAN DEFAULT false,
ADD COLUMN tenant_role VARCHAR(50) DEFAULT 'member';

-- Add tenant isolation to all tables
ALTER TABLE brands ADD COLUMN account_id UUID REFERENCES accounts(id);
ALTER TABLE partnerships ADD COLUMN account_id UUID REFERENCES accounts(id);
ALTER TABLE member_profiles ADD COLUMN account_id UUID REFERENCES accounts(id);

-- Row-level security policies
ALTER TABLE brands ENABLE ROW LEVEL SECURITY;
ALTER TABLE partnerships ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation_brands ON brands
    FOR ALL USING (account_id = current_setting('app.current_account')::UUID);

CREATE POLICY tenant_isolation_partnerships ON partnerships
    FOR ALL USING (account_id = current_setting('app.current_account')::UUID);
```

## Implementation Steps

### Phase 1: Repository Setup (Day 1)
1. Create directory structure in each repository
2. Initialize package.json/requirements.txt
3. Set up Docker configurations
4. Create base CI/CD pipelines

### Phase 2: Code Migration (Day 2-3)
1. Move services to appropriate repositories
2. Update import paths and dependencies
3. Create inter-service communication interfaces
4. Set up shared types/interfaces package

### Phase 3: Multi-Tenant Implementation (Day 3-4)
1. Implement account/organization models
2. Add tenant context to JWT tokens
3. Implement row-level security
4. Create admin interfaces

### Phase 4: Kong Gateway Setup (Day 4)
1. Deploy Kong with PostgreSQL
2. Configure all service routes
3. Set up authentication plugins
4. Configure rate limiting

### Phase 5: Azure Deployment (Day 5)
1. Create Azure resources via Bicep
2. Build and push Docker images
3. Deploy Container Apps
4. Configure DNS for teraffi.io

### Phase 6: Testing & Monitoring (Day 6)
1. End-to-end testing
2. Set up monitoring dashboards
3. Configure alerts
4. Performance optimization

## Docker Compose Files

### Production Compose (docker-compose.production.yml)
```yaml
version: '3.8'

services:
  # Kong Gateway
  kong:
    image: kong:3.4
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: postgres
      KONG_PG_DATABASE: kong
      KONG_PROXY_ACCESS_LOG: /dev/stdout
      KONG_ADMIN_ACCESS_LOG: /dev/stdout
      KONG_PROXY_ERROR_LOG: /dev/stderr
      KONG_ADMIN_ERROR_LOG: /dev/stderr
      KONG_ADMIN_LISTEN: 0.0.0.0:8001
    ports:
      - "80:8000"
      - "443:8443"
      - "8001:8001"
    depends_on:
      - postgres
    networks:
      - teraffi-network

  # Platform Services
  auth-service:
    image: teraffi.azurecr.io/platform/auth:latest
    environment:
      DATABASE_URL: postgresql://user:pass@postgres:5432/teraffi
      JWT_SECRET: ${JWT_SECRET}
      REDIS_URL: redis://redis:6379
    networks:
      - teraffi-network

  partnership-service:
    image: teraffi.azurecr.io/platform/partnerships:latest
    environment:
      DATABASE_URL: postgresql://user:pass@postgres:5432/teraffi
      NEO4J_URL: bolt://neo4j:7687
    networks:
      - teraffi-network

  # AI Services  
  graphrag-service:
    image: teraffi.azurecr.io/ai/graphrag:latest
    environment:
      NEO4J_URL: bolt://neo4j:7687
      PGVECTOR_URL: postgresql://user:pass@postgres:5432/embeddings
      REDIS_URL: redis://redis:6380
    networks:
      - teraffi-network

  llm-gateway:
    image: teraffi.azurecr.io/ai/llm-gateway:latest
    environment:
      OLLAMA_BASE_URL: http://ollama:11434
      OPENROUTER_API_KEY: ${OPENROUTER_API_KEY}
      USE_OPENROUTER: true
    networks:
      - teraffi-network

  # Databases
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_MULTIPLE_DATABASES: teraffi,kong,embeddings
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - teraffi-network

  neo4j:
    image: neo4j:5.13.0
    environment:
      NEO4J_AUTH: neo4j/${NEO4J_PASSWORD}
      NEO4J_dbms_memory_heap_max__size: 2G
      NEO4J_dbms_memory_pagecache_size: 1G
    volumes:
      - neo4j-data:/data
    networks:
      - teraffi-network

  redis:
    image: redis:7-alpine
    command: redis-server --port 6379
    volumes:
      - redis-data:/data
    networks:
      - teraffi-network

  redis-graphrag:
    image: redis:7-alpine
    command: redis-server --port 6380
    volumes:
      - redis-graphrag-data:/data
    networks:
      - teraffi-network

  ollama:
    image: ollama/ollama:latest
    command: serve
    volumes:
      - ollama-data:/root/.ollama
    networks:
      - teraffi-network

networks:
  teraffi-network:
    driver: bridge

volumes:
  postgres-data:
  neo4j-data:
  redis-data:
  redis-graphrag-data:
  ollama-data:
```

## CI/CD Pipeline (GitHub Actions)

### Deploy to Azure (.github/workflows/deploy-azure.yml)
```yaml
name: Deploy to Azure

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Login to Azure
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      
      - name: Login to ACR
        uses: azure/docker-login@v1
        with:
          login-server: teraffi.azurecr.io
          username: ${{ secrets.ACR_USERNAME }}
          password: ${{ secrets.ACR_PASSWORD }}
      
      - name: Build and Push Images
        run: |
          docker-compose -f docker-compose.production.yml build
          docker-compose -f docker-compose.production.yml push
      
      - name: Deploy to Container Apps
        uses: azure/container-apps-deploy-action@v1
        with:
          resourceGroup: teraffi-rg
          containerAppName: teraffi-platform
          yamlConfigPath: ./azure/container-apps.yaml
```

## Monitoring & Health Checks

### Health Check Endpoints
Each service exposes:
- `GET /health` - Basic health status
- `GET /health/detailed` - Database connectivity, memory usage
- `GET /ready` - Readiness for traffic

### Monitoring Dashboard
- Grafana for visualization
- Prometheus for metrics collection
- Azure Log Analytics for centralized logging
- Custom dashboards for business metrics

## Security Implementation

### Authentication Flow
1. User logs in via `/api/v1/auth/login`
2. JWT token issued with tenant context
3. Kong validates JWT on every request
4. Services extract tenant from token
5. Database queries filtered by tenant

### Network Security
- All services in private network
- Only Kong exposed publicly
- SSL/TLS encryption for all traffic
- WAF protection via Application Gateway

## Cost Optimization Strategies

1. **Container Apps Scaling**
   - Scale to zero during off-hours
   - Auto-scale based on CPU/memory
   - Use consumption plan for dev/staging

2. **Database Optimization**
   - Use Basic tier for dev/staging
   - Schedule backups during off-peak
   - Implement connection pooling

3. **Storage Optimization**
   - Use blob storage for large files
   - Implement CDN for static assets
   - Archive old data to cool storage

## Success Criteria

- [ ] All services deployed and accessible via Kong
- [ ] Multi-tenant isolation working correctly
- [ ] Two-tier admin system functional
- [ ] GraphRAG queries returning results
- [ ] Frontend authenticated and role-based
- [ ] teraffi.io domain configured and SSL working
- [ ] Monitoring dashboards operational
- [ ] Total monthly cost under $250

## Next Steps

1. Begin repository setup and code migration
2. Implement multi-tenant architecture
3. Configure Kong Gateway
4. Deploy to Azure
5. Configure DNS for teraffi.io
6. Run integration tests
7. Set up monitoring