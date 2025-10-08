# ADR-025: Deployment & CI/CD Pipeline

**Status:** Proposed  
**Date:** 2025-10-01  
**Decision Makers:** Platform Architecture Team  
**Related ADRs:** ADR-001 (Runtime), ADR-024 (Observability)

---

## Context

TERAFFI requires a robust deployment pipeline that enables rapid, safe releases while maintaining high availability. The team must be able to deploy multiple times per day with confidence, automated testing, and easy rollback capabilities.

**Deployment Requirements:**

**Environments:**
- **Development**: Local developer machines
- **Staging**: Pre-production testing environment
- **Production**: Customer-facing environment
- **Hotfix**: Emergency production patches

**CI/CD Pipeline:**
- Automated testing on every commit
- Build and package artifacts
- Deploy to staging automatically
- Deploy to production with approval
- Blue-green or canary deployments
- Automated rollback on failures

**Infrastructure as Code:**
- Version-controlled infrastructure
- Reproducible environments
- Disaster recovery capabilities
- Multi-region support (future)

**Release Management:**
- Semantic versioning
- Release notes generation
- Database migrations
- Feature flags
- Gradual rollouts

### Current Gap

Without proper CI/CD:
- **Manual Deployments**: Error-prone, slow, stressful
- **No Automation**: Tests not run consistently
- **Deployment Fear**: Infrequent releases due to risk
- **Slow Rollbacks**: Manual process takes hours
- **Environment Drift**: Dev/staging/prod divergence
- **No Audit Trail**: Unclear what's deployed when

### Requirements

**Functional:**
1. Automated build and test on every commit
2. Containerized deployments (Docker)
3. Infrastructure as Code (Terraform)
4. Database migration management
5. Blue-green or canary deployments
6. Automated rollback on failures
7. Secrets management
8. Environment parity

**Non-Functional:**
1. Build time <10 minutes
2. Deployment time <15 minutes
3. Zero-downtime deployments
4. Rollback time <5 minutes
5. 99.9% deployment success rate
6. Support 20+ deployments per day

---

## Decision

**Implement modern CI/CD pipeline** with:
1. **GitHub Actions** for CI/CD orchestration
2. **Docker** for containerization
3. **Azure Container Registry** (ACR) for image storage
4. **Azure Kubernetes Service** (AKS) for orchestration
5. **Terraform** for infrastructure as code
6. **Helm** for Kubernetes package management
7. **Azure Key Vault** for secrets management
8. **Flyway** for database migrations
9. **Feature flags** (LaunchDarkly or custom)

**Architecture:**
```
Git Push → GitHub
    ↓
[GitHub Actions]
    ↓
Run Tests (unit, integration, e2e)
    ↓
Build Docker Images
    ↓
Push to Azure Container Registry
    ↓
Update Helm Charts
    ↓
Deploy to Staging (automatic)
    ↓
Run Smoke Tests
    ↓
Manual Approval for Production
    ↓
Deploy to Production (blue-green)
    ↓
Health Checks
    ↓
Switch Traffic (if healthy)
    ↓
Monitor & Alert
```

---

## Rationale

### Why GitHub Actions

**Benefits:**
- Native integration with GitHub
- Free for open source, generous free tier
- YAML-based configuration
- Matrix builds for parallel testing
- Self-hosted runners option
- Large marketplace of actions

**Alternatives:**
- **Jenkins**: Self-hosted, more powerful but complex
- **CircleCI**: Good but another service to manage
- **GitLab CI**: Would require GitLab migration
- **Decision**: GitHub Actions for simplicity and integration

### Why Kubernetes

**Benefits:**
- Industry standard for container orchestration
- Zero-downtime deployments
- Auto-scaling
- Self-healing
- Service discovery
- Rolling updates and rollbacks

**Alternatives:**
- **Docker Compose**: Too simple for production
- **Azure Container Instances**: Limited orchestration
- **VMs with systemd**: Manual, hard to scale
- **Decision**: Kubernetes for production-grade orchestration

### Why Blue-Green Deployments

**Benefits:**
- Zero downtime
- Easy rollback (switch traffic back)
- Full environment testing before cutover
- Reduced risk

**Alternatives:**
- **Rolling updates**: Gradual but mixed versions running
- **Canary**: Gradual but complex traffic splitting
- **Decision**: Blue-green for simplicity and safety

### Why Terraform

**Benefits:**
- Infrastructure as code
- Declarative configuration
- State management
- Multi-cloud support
- Large provider ecosystem

**Alternatives:**
- **ARM Templates**: Azure-only, verbose
- **Pulumi**: Code-based, less mature
- **CDK**: Good but newer
- **Decision**: Terraform for maturity and community

---

## Implementation Details

### 1. Project Structure

```
teraffi/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── deploy-staging.yml
│       ├── deploy-production.yml
│       └── rollback.yml
├── apps/
│   ├── api/
│   │   ├── Dockerfile
│   │   └── package.json
│   ├── web/
│   │   ├── Dockerfile
│   │   └── package.json
│   └── worker/
│       ├── Dockerfile
│       └── package.json
├── infrastructure/
│   ├── terraform/
│   │   ├── environments/
│   │   │   ├── staging/
│   │   │   └── production/
│   │   ├── modules/
│   │   │   ├── aks/
│   │   │   ├── database/
│   │   │   ├── redis/
│   │   │   └── networking/
│   │   └── main.tf
│   ├── kubernetes/
│   │   ├── base/
│   │   ├── staging/
│   │   └── production/
│   └── helm/
│       └── teraffi/
│           ├── Chart.yaml
│           ├── values.yaml
│           ├── values-staging.yaml
│           ├── values-production.yaml
│           └── templates/
├── db/
│   └── migrations/
│       └── V001__initial_schema.sql
└── package.json
```

### 2. GitHub Actions - CI Pipeline

```yaml
# .github/workflows/ci.yml

name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '20'
  POSTGRES_VERSION: '15'

jobs:
  lint:
    name: Lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run ESLint
        run: npm run lint
      
      - name: Run Prettier
        run: npm run format:check
      
      - name: TypeScript type check
        run: npm run type-check

  test-unit:
    name: Unit Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run unit tests
        run: npm run test:unit -- --coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info

  test-integration:
    name: Integration Tests
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: teraffi_test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
      
      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run migrations
        run: npm run db:migrate
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/teraffi_test
      
      - name: Run integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/teraffi_test
          REDIS_URL: redis://localhost:6379

  test-e2e:
    name: E2E Tests
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Start test environment
        run: docker-compose -f docker-compose.test.yml up -d
      
      - name: Wait for services
        run: |
          npm run wait-for -- http://localhost:3000/health
          npm run wait-for -- http://localhost:5432
      
      - name: Run E2E tests
        run: npm run test:e2e
      
      - name: Upload test artifacts
        if: failure()
        uses: actions/upload-artifact@v3
        with:
          name: e2e-screenshots
          path: tests/e2e/screenshots/

  build:
    name: Build Docker Images
    runs-on: ubuntu-latest
    needs: [lint, test-unit, test-integration, test-e2e]
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to Azure Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ secrets.ACR_REGISTRY }}
          username: ${{ secrets.ACR_USERNAME }}
          password: ${{ secrets.ACR_PASSWORD }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ secrets.ACR_REGISTRY }}/teraffi-api
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=sha,prefix={{branch}}-
            type=semver,pattern={{version}}
      
      - name: Build and push API
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./apps/api/Dockerfile
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=registry,ref=${{ secrets.ACR_REGISTRY }}/teraffi-api:buildcache
          cache-to: type=registry,ref=${{ secrets.ACR_REGISTRY }}/teraffi-api:buildcache,mode=max
      
      - name: Build and push Web
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./apps/web/Dockerfile
          push: true
          tags: ${{ secrets.ACR_REGISTRY }}/teraffi-web:${{ github.sha }}
      
      - name: Build and push Worker
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./apps/worker/Dockerfile
          push: true
          tags: ${{ secrets.ACR_REGISTRY }}/teraffi-worker:${{ github.sha }}
```

### 3. Deployment Pipeline - Staging

```yaml
# .github/workflows/deploy-staging.yml

name: Deploy to Staging

on:
  push:
    branches: [develop]
  workflow_dispatch:

jobs:
  deploy:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    environment: staging
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup kubectl
        uses: azure/setup-kubectl@v3
      
      - name: Setup Helm
        uses: azure/setup-helm@v3
      
      - name: Login to Azure
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      
      - name: Get AKS credentials
        run: |
          az aks get-credentials \
            --resource-group teraffi-staging \
            --name teraffi-staging-aks
      
      - name: Run database migrations
        run: |
          kubectl run flyway-migration \
            --image=${{ secrets.ACR_REGISTRY }}/teraffi-api:${{ github.sha }} \
            --restart=Never \
            --env="DATABASE_URL=${{ secrets.STAGING_DATABASE_URL }}" \
            --command -- npm run db:migrate
          
          kubectl wait --for=condition=complete job/flyway-migration --timeout=5m
      
      - name: Deploy with Helm
        run: |
          helm upgrade --install teraffi ./infrastructure/helm/teraffi \
            --namespace staging \
            --create-namespace \
            --values ./infrastructure/helm/teraffi/values-staging.yaml \
            --set image.tag=${{ github.sha }} \
            --set image.repository=${{ secrets.ACR_REGISTRY }} \
            --wait \
            --timeout 10m
      
      - name: Run smoke tests
        run: |
          export STAGING_URL="https://staging.teraffi.io"
          npm run test:smoke
      
      - name: Notify deployment
        if: always()
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Staging deployment ${{ job.status }}'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

### 4. Deployment Pipeline - Production

```yaml
# .github/workflows/deploy-production.yml

name: Deploy to Production

on:
  push:
    tags:
      - 'v*.*.*'
  workflow_dispatch:
    inputs:
      version:
        description: 'Version to deploy'
        required: true

jobs:
  deploy:
    name: Deploy to Production
    runs-on: ubuntu-latest
    environment: 
      name: production
      url: https://teraffi.io
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup kubectl
        uses: azure/setup-kubectl@v3
      
      - name: Setup Helm
        uses: azure/setup-helm@v3
      
      - name: Login to Azure
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      
      - name: Get AKS credentials
        run: |
          az aks get-credentials \
            --resource-group teraffi-production \
            --name teraffi-production-aks
      
      - name: Create blue-green namespace
        run: |
          DEPLOY_ENV="green"
          if kubectl get namespace production-blue; then
            DEPLOY_ENV="green"
          else
            DEPLOY_ENV="blue"
          fi
          echo "DEPLOY_ENV=$DEPLOY_ENV" >> $GITHUB_ENV
          kubectl create namespace production-$DEPLOY_ENV --dry-run=client -o yaml | kubectl apply -f -
      
      - name: Run database migrations
        run: |
          kubectl run flyway-migration-${{ github.run_id }} \
            --image=${{ secrets.ACR_REGISTRY }}/teraffi-api:${{ github.sha }} \
            --restart=Never \
            --env="DATABASE_URL=${{ secrets.PRODUCTION_DATABASE_URL }}" \
            --command -- npm run db:migrate
          
          kubectl wait --for=condition=complete job/flyway-migration-${{ github.run_id }} --timeout=10m
      
      - name: Deploy to blue-green environment
        run: |
          helm upgrade --install teraffi-${{ env.DEPLOY_ENV }} ./infrastructure/helm/teraffi \
            --namespace production-${{ env.DEPLOY_ENV }} \
            --values ./infrastructure/helm/teraffi/values-production.yaml \
            --set image.tag=${{ github.sha }} \
            --set image.repository=${{ secrets.ACR_REGISTRY }} \
            --set environment=${{ env.DEPLOY_ENV }} \
            --wait \
            --timeout 15m
      
      - name: Wait for deployment to be ready
        run: |
          kubectl wait --for=condition=available --timeout=10m \
            deployment/teraffi-api -n production-${{ env.DEPLOY_ENV }}
      
      - name: Run health checks
        run: |
          export API_URL="https://api-${{ env.DEPLOY_ENV }}.teraffi.io"
          npm run health-check
      
      - name: Run smoke tests
        run: |
          export API_URL="https://api-${{ env.DEPLOY_ENV }}.teraffi.io"
          npm run test:smoke
      
      - name: Switch traffic to new environment
        run: |
          kubectl patch service teraffi-api-public \
            -n production \
            -p '{"spec":{"selector":{"environment":"${{ env.DEPLOY_ENV }}"}}}'
      
      - name: Monitor for 5 minutes
        run: |
          sleep 300
          # Check error rates
          ERROR_RATE=$(curl -s "https://prometheus.teraffi.io/api/v1/query?query=rate(http_requests_total{status_code=~\"5..\"}[5m])" | jq '.data.result[0].value[1]')
          if (( $(echo "$ERROR_RATE > 0.05" | bc -l) )); then
            echo "High error rate detected: $ERROR_RATE"
            exit 1
          fi
      
      - name: Rollback on failure
        if: failure()
        run: |
          OLD_ENV="blue"
          if [ "${{ env.DEPLOY_ENV }}" == "blue" ]; then
            OLD_ENV="green"
          fi
          
          kubectl patch service teraffi-api-public \
            -n production \
            -p '{"spec":{"selector":{"environment":"'$OLD_ENV'"}}}'
          
          echo "Rolled back to $OLD_ENV environment"
      
      - name: Create release notes
        if: success()
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          body: |
            Deployed to production
            
            **Changes:**
            ${{ github.event.head_commit.message }}
          draft: false
          prerelease: false
      
      - name: Notify deployment
        if: always()
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Production deployment ${{ job.status }} - ${{ env.DEPLOY_ENV }}'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

### 5. Dockerfile Example

```dockerfile
# apps/api/Dockerfile

# Build stage
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./
COPY turbo.json ./

# Copy workspace packages
COPY apps/api/package*.json ./apps/api/
COPY packages/*/package*.json ./packages/

# Install dependencies
RUN npm ci

# Copy source code
COPY apps/api ./apps/api
COPY packages ./packages

# Build
RUN npm run build --workspace=apps/api

# Prune dev dependencies
RUN npm prune --production

# Production stage
FROM node:20-alpine

WORKDIR /app

# Install dumb-init for proper signal handling
RUN apk add --no-cache dumb-init

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Copy built application
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/apps/api/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/apps/api/package.json ./

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

USER nodejs

EXPOSE 3000

# Use dumb-init to handle signals properly
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "dist/main.js"]
```

### 6. Helm Chart

```yaml
# infrastructure/helm/teraffi/values.yaml

replicaCount: 3

image:
  repository: teraffi.azurecr.io/teraffi-api
  pullPolicy: IfNotPresent
  tag: "latest"

imagePullSecrets:
  - name: acr-secret

service:
  type: ClusterIP
  port: 80
  targetPort: 3000

ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rate-limit: "100"
  hosts:
    - host: api.teraffi.io
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: api-tls
      hosts:
        - api.teraffi.io

resources:
  limits:
    cpu: 1000m
    memory: 1Gi
  requests:
    cpu: 250m
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 20
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80

livenessProbe:
  httpGet:
    path: /health
    port: 3000
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 3000
  initialDelaySeconds: 10
  periodSeconds: 5

env:
  - name: NODE_ENV
    value: "production"
  - name: DATABASE_URL
    valueFrom:
      secretKeyRef:
        name: database-credentials
        key: url
  - name: REDIS_URL
    valueFrom:
      secretKeyRef:
        name: redis-credentials
        key: url

podDisruptionBudget:
  enabled: true
  minAvailable: 2

nodeSelector: {}

tolerations: []

affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
              - key: app
                operator: In
                values:
                  - teraffi-api
          topologyKey: kubernetes.io/hostname
```

### 7. Terraform Infrastructure

```hcl
# infrastructure/terraform/environments/production/main.tf

terraform {
  required_version = ">= 1.5"
  
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
  
  backend "azurerm" {
    resource_group_name  = "teraffi-terraform-state"
    storage_account_name = "teraffitfstate"
    container_name       = "tfstate"
    key                  = "production.terraform.tfstate"
  }
}

provider "azurerm" {
  features {}
}

# Resource Group
resource "azurerm_resource_group" "main" {
  name     = "teraffi-production"
  location = "East US"
  
  tags = {
    Environment = "production"
    ManagedBy   = "terraform"
  }
}

# Virtual Network
module "networking" {
  source = "../../modules/networking"
  
  resource_group_name = azurerm_resource_group.main.name
  location           = azurerm_resource_group.main.location
  environment        = "production"
}

# AKS Cluster
module "aks" {
  source = "../../modules/aks"
  
  resource_group_name = azurerm_resource_group.main.name
  location           = azurerm_resource_group.main.location
  environment        = "production"
  
  vnet_subnet_id = module.networking.aks_subnet_id
  
  node_count         = 5
  node_size          = "Standard_D4s_v3"
  min_nodes          = 3
  max_nodes          = 20
}

# PostgreSQL
module "database" {
  source = "../../modules/database"
  
  resource_group_name = azurerm_resource_group.main.name
  location           = azurerm_resource_group.main.location
  environment        = "production"
  
  sku_name           = "GP_Gen5_4"
  storage_mb         = 102400
  backup_retention_days = 35
  geo_redundant_backup  = true
}

# Redis Cache
module "redis" {
  source = "../../modules/redis"
  
  resource_group_name = azurerm_resource_group.main.name
  location           = azurerm_resource_group.main.location
  environment        = "production"
  
  sku_name = "Premium"
  capacity = 2
}

# Key Vault
resource "azurerm_key_vault" "main" {
  name                = "teraffi-prod-kv"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  tenant_id           = data.azurerm_client_config.current.tenant_id
  sku_name            = "standard"
  
  purge_protection_enabled = true
  
  network_acls {
    default_action = "Deny"
    bypass         = "AzureServices"
  }
}

# Container Registry
resource "azurerm_container_registry" "main" {
  name                = "teraffiacr"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  sku                 = "Premium"
  admin_enabled       = false
  
  georeplications {
    location = "West US"
  }
}

# Outputs
output "aks_cluster_name" {
  value = module.aks.cluster_name
}

output "database_fqdn" {
  value     = module.database.fqdn
  sensitive = true
}

output "redis_hostname" {
  value     = module.redis.hostname
  sensitive = true
}
```

---

## Migration Path

### Phase 1: Foundation (Months 1-2)
- Set up GitHub Actions CI
- Containerize applications (Docker)
- Basic automated testing

### Phase 2: Infrastructure (Months 3-4)
- Terraform infrastructure as code
- AKS cluster deployment
- Database and Redis setup

### Phase 3: CD Pipeline (Months 5-6)
- Helm charts
- Staging auto-deployment
- Production deployment with approval

### Phase 4: Advanced (Months 7-8)
- Blue-green deployments
- Canary releases
- Automated rollbacks
- Feature flags

---

## Success Metrics

**Deployment Frequency:**
- 10+ deploys per day to staging
- 5+ deploys per day to production
- Deploy on every merge to main

**Lead Time:**
- Commit to production <2 hours
- Build time <10 minutes
- Deployment time <15 minutes

**Reliability:**
- 99.9% deployment success rate
- <5 minute rollback time
- Zero-downtime deployments

**Quality:**
- 0 failed deployments in production
- <1% rollback rate
- 100% passing tests before deploy

---

## References

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Helm Documentation](https://helm.sh/docs/)
- [Terraform Azure Provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs)
- [The Twelve-Factor App](https://12factor.net/)

---

**Next Steps:**
1. Review and approve ADR-025
2. Create task breakdown (ADR-025_Task_001 through ADR-025_Task_N)
3. Set up GitHub Actions
4. Create Dockerfiles
5. Deploy staging environment