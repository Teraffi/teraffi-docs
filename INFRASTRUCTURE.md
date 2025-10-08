# TERAFFI Infrastructure & Deployment Guide
## Production-Ready Kubernetes Deployment on AWS

---

## Infrastructure Overview

TERAFFI's infrastructure is designed for high availability, scalability, and cost efficiency using modern cloud-native technologies.

### Architecture Summary
- **Orchestration**: Amazon EKS (Kubernetes)
- **Load Balancing**: AWS Application Load Balancer + Nginx Ingress
- **Databases**: RDS PostgreSQL, Neo4j AuraDB, ElastiCache Redis
- **Storage**: S3 for assets, EFS for shared storage
- **Monitoring**: CloudWatch, DataDog, Prometheus/Grafana
- **CI/CD**: GitHub Actions, ArgoCD
- **Security**: IAM, VPC, Security Groups, WAF

---

## AWS Infrastructure Setup

### VPC and Networking

```yaml
# vpc.yaml - Virtual Private Cloud Configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: vpc-config
data:
  vpc_cidr: "10.0.0.0/16"
  public_subnets:
    - "10.0.1.0/24"  # us-west-2a
    - "10.0.2.0/24"  # us-west-2b
    - "10.0.3.0/24"  # us-west-2c
  private_subnets:
    - "10.0.11.0/24" # us-west-2a
    - "10.0.12.0/24" # us-west-2b
    - "10.0.13.0/24" # us-west-2c
  database_subnets:
    - "10.0.21.0/24" # us-west-2a
    - "10.0.22.0/24" # us-west-2b
    - "10.0.23.0/24" # us-west-2c
```

### EKS Cluster Configuration

```yaml
# eks-cluster.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: teraffi-production
  region: us-west-2
  version: "1.28"

vpc:
  cidr: "10.0.0.0/16"
  nat:
    gateway: HighlyAvailable

managedNodeGroups:
  - name: api-services
    instanceType: m5.large
    desiredCapacity: 3
    minSize: 2
    maxSize: 10
    volumeSize: 100
    volumeType: gp3
    labels:
      role: api-services
    taints:
      - key: role
        value: api-services
        effect: NoSchedule
    iam:
      withAddonPolicies:
        autoScaler: true
        cloudWatch: true
        ebs: true
        efs: true
        albIngress: true

  - name: ai-processing
    instanceType: c5.2xlarge
    desiredCapacity: 2
    minSize: 1
    maxSize: 8
    volumeSize: 200
    volumeType: gp3
    labels:
      role: ai-processing
    taints:
      - key: role
        value: ai-processing
        effect: NoSchedule

  - name: background-jobs
    instanceType: m5.xlarge
    desiredCapacity: 2
    minSize: 1
    maxSize: 6
    volumeSize: 100
    volumeType: gp3
    labels:
      role: background-jobs
    spot: true # Cost optimization for non-critical workloads

addons:
  - name: vpc-cni
    version: latest
  - name: coredns
    version: latest
  - name: kube-proxy
    version: latest
  - name: aws-ebs-csi-driver
    version: latest
  - name: aws-load-balancer-controller
    version: latest

iam:
  withOIDC: true
  serviceAccounts:
    - metadata:
        name: aws-load-balancer-controller
        namespace: kube-system
      wellKnownPolicies:
        awsLoadBalancerController: true
    - metadata:
        name: cluster-autoscaler
        namespace: kube-system
      wellKnownPolicies:
        autoScaler: true
```

---

## Database Infrastructure

### RDS PostgreSQL Configuration

```yaml
# rds-postgresql.yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgresql-credentials
type: Opaque
data:
  username: cG9zdGdyZXM=  # postgres (base64)
  password: <base64-encoded-password>
  database: dGVyYWZmaQ==   # teraffi (base64)

---
# PostgreSQL RDS configuration (via Terraform/CloudFormation)
resource "aws_db_instance" "postgresql_primary" {
  identifier = "teraffi-postgres-primary"
  
  # Engine
  engine         = "postgres"
  engine_version = "15.4"
  
  # Instance
  instance_class = "db.r5.xlarge"
  allocated_storage = 1000
  max_allocated_storage = 5000
  storage_type = "gp3"
  storage_encrypted = true
  
  # Database
  db_name  = "teraffi"
  username = "postgres"
  password = var.db_password
  
  # Networking
  db_subnet_group_name = aws_db_subnet_group.database.name
  vpc_security_group_ids = [aws_security_group.database.id]
  publicly_accessible = false
  
  # Backup and Maintenance
  backup_retention_period = 30
  backup_window = "03:00-04:00"
  maintenance_window = "sun:04:00-sun:05:00"
  
  # High Availability
  multi_az = true
  
  # Performance
  performance_insights_enabled = true
  monitoring_interval = 60
  
  # Deletion Protection
  deletion_protection = true
  skip_final_snapshot = false
  final_snapshot_identifier = "teraffi-postgres-final-snapshot"
  
  tags = {
    Name = "teraffi-postgres-primary"
    Environment = "production"
  }
}

# Read Replica for Analytics
resource "aws_db_instance" "postgresql_replica" {
  identifier = "teraffi-postgres-replica"
  replicate_source_db = aws_db_instance.postgresql_primary.id
  instance_class = "db.r5.large"
  publicly_accessible = false
  
  tags = {
    Name = "teraffi-postgres-replica"
    Environment = "production"
  }
}
```

### Neo4j AuraDB Configuration

```yaml
# neo4j-connection.yaml
apiVersion: v1
kind: Secret
metadata:
  name: neo4j-credentials
type: Opaque
data:
  uri: <base64-encoded-aura-uri>
  username: <base64-encoded-username>
  password: <base64-encoded-password>

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: neo4j-config
data:
  instance_size: "8GB"
  cluster_type: "enterprise"
  backup_frequency: "daily"
  region: "us-west-2"
```

### Redis ElastiCache Configuration

```yaml
# redis-elasticache.yaml
resource "aws_elasticache_subnet_group" "redis" {
  name       = "teraffi-redis-subnet-group"
  subnet_ids = var.private_subnet_ids
}

resource "aws_elasticache_replication_group" "redis" {
  replication_group_id       = "teraffi-redis"
  description                = "Redis cluster for TERAFFI"
  
  node_type                  = "cache.r5.large"
  port                       = 6379
  parameter_group_name       = "default.redis7"
  
  num_cache_clusters         = 3
  automatic_failover_enabled = true
  multi_az_enabled          = true
  
  subnet_group_name = aws_elasticache_subnet_group.redis.name
  security_group_ids = [aws_security_group.redis.id]
  
  at_rest_encryption_enabled = true
  transit_encryption_enabled = true
  
  maintenance_window = "sun:05:00-sun:06:00"
  snapshot_retention_limit = 7
  snapshot_window = "03:00-05:00"
  
  log_delivery_configuration {
    destination      = aws_cloudwatch_log_group.redis.name
    destination_type = "cloudwatch-logs"
    log_format      = "text"
    log_type        = "slow-log"
  }
  
  tags = {
    Name = "teraffi-redis"
    Environment = "production"
  }
}
```

---

## Kubernetes Application Deployment

### Namespace Configuration

```yaml
# namespaces.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: teraffi-production
  labels:
    name: teraffi-production
    environment: production

---
apiVersion: v1
kind: Namespace
metadata:
  name: teraffi-monitoring
  labels:
    name: teraffi-monitoring
    environment: production

---
apiVersion: v1
kind: Namespace
metadata:
  name: teraffi-ingress
  labels:
    name: teraffi-ingress
    environment: production
```

### Core Services Deployment

#### API Gateway Service

```yaml
# api-gateway-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
  namespace: teraffi-production
  labels:
    app: api-gateway
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: api-gateway
  template:
    metadata:
      labels:
        app: api-gateway
    spec:
      containers:
      - name: api-gateway
        image: teraffi/api-gateway:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: DB_HOST
          valueFrom:
            secretKeyRef:
              name: postgresql-credentials
              key: host
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgresql-credentials
              key: password
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: redis-credentials
              key: url
        - name: NEO4J_URI
          valueFrom:
            secretKeyRef:
              name: neo4j-credentials
              key: uri
        resources:
          requests:
            cpu: 200m
            memory: 512Mi
          limits:
            cpu: 500m
            memory: 1Gi
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
          initialDelaySeconds: 5
          periodSeconds: 5
      tolerations:
      - key: role
        value: api-services
        operator: Equal
        effect: NoSchedule
      nodeSelector:
        role: api-services

---
apiVersion: v1
kind: Service
metadata:
  name: api-gateway-service
  namespace: teraffi-production
spec:
  selector:
    app: api-gateway
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  type: ClusterIP

---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-gateway-hpa
  namespace: teraffi-production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-gateway
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

#### Affinity Engine Service

```yaml
# affinity-engine-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: affinity-engine
  namespace: teraffi-production
  labels:
    app: affinity-engine
spec:
  replicas: 2
  selector:
    matchLabels:
      app: affinity-engine
  template:
    metadata:
      labels:
        app: affinity-engine
    spec:
      containers:
      - name: affinity-engine
        image: teraffi/affinity-engine:latest
        ports:
        - containerPort: 8000
        env:
        - name: PYTHON_ENV
          value: "production"
        - name: NEO4J_URI
          valueFrom:
            secretKeyRef:
              name: neo4j-credentials
              key: uri
        - name: PINECONE_API_KEY
          valueFrom:
            secretKeyRef:
              name: pinecone-credentials
              key: api_key
        - name: OPENAI_API_KEY
          valueFrom:
            secretKeyRef:
              name: openai-credentials
              key: api_key
        resources:
          requests:
            cpu: 1000m
            memory: 2Gi
          limits:
            cpu: 2000m
            memory: 4Gi
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 60
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /ready
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
      tolerations:
      - key: role
        value: ai-processing
        operator: Equal
        effect: NoSchedule
      nodeSelector:
        role: ai-processing

---
apiVersion: v1
kind: Service
metadata:
  name: affinity-engine-service
  namespace: teraffi-production
spec:
  selector:
    app: affinity-engine
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8000
  type: ClusterIP

---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: affinity-engine-hpa
  namespace: teraffi-production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: affinity-engine
  minReplicas: 2
  maxReplicas: 8
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 75
```

#### Background Job Processor

```yaml
# background-jobs-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: background-jobs
  namespace: teraffi-production
  labels:
    app: background-jobs
spec:
  replicas: 2
  selector:
    matchLabels:
      app: background-jobs
  template:
    metadata:
      labels:
        app: background-jobs
    spec:
      containers:
      - name: job-processor
        image: teraffi/job-processor:latest
        env:
        - name: NODE_ENV
          value: "production"
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: redis-credentials
              key: url
        - name: DB_HOST
          valueFrom:
            secretKeyRef:
              name: postgresql-credentials
              key: host
        resources:
          requests:
            cpu: 500m
            memory: 1Gi
          limits:
            cpu: 1000m
            memory: 2Gi
      tolerations:
      - key: role
        value: background-jobs
        operator: Equal
        effect: NoSchedule
      nodeSelector:
        role: background-jobs

---
apiVersion: v1
kind: Service
metadata:
  name: background-jobs-service
  namespace: teraffi-production
spec:
  selector:
    app: background-jobs
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  type: ClusterIP
```

---

## Ingress and Load Balancing

### Nginx Ingress Controller

```yaml
# ingress-controller.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ingress-nginx

---
apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: nginx-ingress
  namespace: kube-system
spec:
  chart: ingress-nginx
  repo: https://kubernetes.github.io/ingress-nginx
  targetNamespace: ingress-nginx
  valuesContent: |-
    controller:
      replicaCount: 3
      service:
        type: LoadBalancer
        annotations:
          service.beta.kubernetes.io/aws-load-balancer-type: nlb
          service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
      metrics:
        enabled: true
      podAnnotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "10254"
      config:
        use-forwarded-headers: "true"
        compute-full-forwarded-for: "true"
        proxy-body-size: "100m"
        client-max-body-size: "100m"
```

### Application Ingress

```yaml
# teraffi-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: teraffi-ingress
  namespace: teraffi-production
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "100m"
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/rate-limit-window: "1m"
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - api.teraffi.com
    secretName: teraffi-api-tls
  rules:
  - host: api.teraffi.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-gateway-service
            port:
              number: 80
      - path: /affinity
        pathType: Prefix
        backend:
          service:
            name: affinity-engine-service
            port:
              number: 80
```

---

## Monitoring and Observability

### Prometheus and Grafana Setup

```yaml
# monitoring-namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring

---
# prometheus-config.yaml
apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: kube-prometheus-stack
  namespace: kube-system
spec:
  chart: kube-prometheus-stack
  repo: https://prometheus-community.github.io/helm-charts
  targetNamespace: monitoring
  valuesContent: |-
    prometheus:
      prometheusSpec:
        retention: 30d
        resources:
          requests:
            cpu: 500m
            memory: 4Gi
          limits:
            cpu: 1000m
            memory: 8Gi
        storageSpec:
          volumeClaimTemplate:
            spec:
              storageClassName: gp3
              resources:
                requests:
                  storage: 100Gi
    grafana:
      adminPassword: <secure-password>
      resources:
        requests:
          cpu: 100m
          memory: 512Mi
        limits:
          cpu: 200m
          memory: 1Gi
      persistence:
        enabled: true
        storageClassName: gp3
        size: 10Gi
    alertmanager:
      alertmanagerSpec:
        resources:
          requests:
            cpu: 100m
            memory: 256Mi
          limits:
            cpu: 200m
            memory: 512Mi
```

### Custom Metrics and Dashboards

```yaml
# teraffi-servicemonitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: teraffi-apps
  namespace: monitoring
  labels:
    app: teraffi
spec:
  selector:
    matchLabels:
      monitoring: "true"
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
  namespaceSelector:
    matchNames:
    - teraffi-production

---
# Custom dashboard ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: teraffi-dashboard
  namespace: monitoring
  labels:
    grafana_dashboard: "1"
data:
  dashboard.json: |
    {
      "dashboard": {
        "title": "TERAFFI Production Metrics",
        "panels": [
          {
            "title": "API Response Time",
            "type": "graph",
            "targets": [
              {
                "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket{service="api-gateway"}[5m]))",
                "legendFormat": "95th percentile"
              }
            ]
          },
          {
            "title": "Affinity Engine Performance",
            "type": "graph",
            "targets": [
              {
                "expr": "rate(affinity_calculations_total[5m])",
                "legendFormat": "Calculations per second"
              }
            ]
          },
          {
            "title": "Partnership Success Rate",
            "type": "singlestat",
            "targets": [
              {
                "expr": "rate(partnerships_successful_total[24h]) / rate(partnerships_created_total[24h])",
                "legendFormat": "Success Rate"
              }
            ]
          }
        ]
      }
    }
```

---

## CI/CD Pipeline

### GitHub Actions Workflow

```yaml
# .github/workflows/deploy-production.yml
name: Deploy to Production

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  AWS_REGION: us-west-2
  ECR_REPOSITORY: teraffi
  EKS_CLUSTER_NAME: teraffi-production

jobs:
  test:
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
      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run unit tests
      run: npm run test:unit
      env:
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/teraffi_test
        REDIS_URL: redis://localhost:6379

    - name: Run integration tests
      run: npm run test:integration
      env:
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/teraffi_test
        REDIS_URL: redis://localhost:6379

    - name: Run security audit
      run: npm audit --audit-level=critical

  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Login to Amazon ECR
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build, tag, and push API Gateway image
      run: |
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY/api-gateway:$GITHUB_SHA ./services/api-gateway
        docker tag $ECR_REGISTRY/$ECR_REPOSITORY/api-gateway:$GITHUB_SHA $ECR_REGISTRY/$ECR_REPOSITORY/api-gateway:latest
        docker push $ECR_REGISTRY/$ECR_REPOSITORY/api-gateway:$GITHUB_SHA
        docker push $ECR_REGISTRY/$ECR_REPOSITORY/api-gateway:latest
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}

    - name: Build, tag, and push Affinity Engine image
      run: |
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY/affinity-engine:$GITHUB_SHA ./services/affinity-engine
        docker tag $ECR_REGISTRY/$ECR_REPOSITORY/affinity-engine:$GITHUB_SHA $ECR_REGISTRY/$ECR_REPOSITORY/affinity-engine:latest
        docker push $ECR_REGISTRY/$ECR_REPOSITORY/affinity-engine:$GITHUB_SHA
        docker push $ECR_REGISTRY/$ECR_REPOSITORY/affinity-engine:latest
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Update kubeconfig
      run: |
        aws eks update-kubeconfig --region $AWS_REGION --name $EKS_CLUSTER_NAME

    - name: Deploy to EKS
      run: |
        # Update image tags in deployment files
        sed -i 's|image: teraffi/api-gateway:latest|image: ${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY/api-gateway:${{ github.sha }}|' k8s/api-gateway-deployment.yaml
        sed -i 's|image: teraffi/affinity-engine:latest|image: ${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY/affinity-engine:${{ github.sha }}|' k8s/affinity-engine-deployment.yaml
        
        # Apply Kubernetes manifests
        kubectl apply -f k8s/namespaces.yaml
        kubectl apply -f k8s/secrets/ -n teraffi-production
        kubectl apply -f k8s/configmaps/ -n teraffi-production
        kubectl apply -f k8s/api-gateway-deployment.yaml
        kubectl apply -f k8s/affinity-engine-deployment.yaml
        kubectl apply -f k8s/background-jobs-deployment.yaml
        kubectl apply -f k8s/ingress.yaml
        
        # Wait for rollout to complete
        kubectl rollout status deployment/api-gateway -n teraffi-production
        kubectl rollout status deployment/affinity-engine -n teraffi-production
        kubectl rollout status deployment/background-jobs -n teraffi-production

    - name: Verify deployment
      run: |
        kubectl get pods -n teraffi-production
        kubectl get services -n teraffi-production
        kubectl get ingress -n teraffi-production
```

### ArgoCD Deployment (GitOps)

```yaml
# argocd-application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: teraffi-production
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/teraffi/infrastructure
    targetRevision: HEAD
    path: k8s/production
  destination:
    server: https://kubernetes.default.svc
    namespace: teraffi-production
  syncPolicy:
    syncOptions:
    - CreateNamespace=true
    automated:
      prune: true
      selfHeal: true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

---

## Security Configuration

### Pod Security Standards

```yaml
# pod-security-policy.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: teraffi-production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: teraffi-service-account
  namespace: teraffi-production

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: teraffi-production
  name: teraffi-role
rules:
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  verbs: ["get", "list"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: teraffi-role-binding
  namespace: teraffi-production
roleRef:
  kind: Role
  name: teraffi-role
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: teraffi-service-account
  namespace: teraffi-production
```

### Network Policies

```yaml
# network-policies.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: teraffi-network-policy
  namespace: teraffi-production
spec:
  podSelector:
    matchLabels:
      app: teraffi
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 3000
    - protocol: TCP
      port: 8000
  - from:
    - podSelector:
        matchLabels:
          app: teraffi
    ports:
    - protocol: TCP
      port: 3000
    - protocol: TCP
      port: 8000
  egress:
  - to: []
    ports:
    - protocol: TCP
      port: 443  # HTTPS
    - protocol: TCP
      port: 5432 # PostgreSQL
    - protocol: TCP
      port: 7687 # Neo4j
    - protocol: TCP
      port: 6379 # Redis
```

---

## Backup and Disaster Recovery

### Database Backup Strategy

```yaml
# backup-cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: postgresql-backup
  namespace: teraffi-production
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: postgres-backup
            image: postgres:15
            env:
            - name: PGPASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgresql-credentials
                  key: password
            command:
            - /bin/bash
            - -c
            - |
              pg_dump -h $DB_HOST -U postgres -d teraffi --format=custom --compress=9 > /backup/teraffi_$(date +\%Y\%m\%d_\%H\%M\%S).dump
              aws s3 cp /backup/teraffi_$(date +\%Y\%m\%d_\%H\%M\%S).dump s3://teraffi-backups/postgresql/
              # Cleanup old backups (keep last 30 days)
              find /backup -name "teraffi_*.dump" -mtime +30 -delete
            volumeMounts:
            - name: backup-storage
              mountPath: /backup
          volumes:
          - name: backup-storage
            persistentVolumeClaim:
              claimName: backup-pvc
          restartPolicy: OnFailure

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: backup-pvc
  namespace: teraffi-production
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Gi
  storageClassName: gp3
```

---

## Cost Optimization

### Resource Optimization

```yaml
# vertical-pod-autoscaler.yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: api-gateway-vpa
  namespace: teraffi-production
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-gateway
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: api-gateway
      minAllowed:
        cpu: 100m
        memory: 256Mi
      maxAllowed:
        cpu: 1000m
        memory: 2Gi
```

### Cluster Autoscaling

```yaml
# cluster-autoscaler.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    app: cluster-autoscaler
spec:
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
    spec:
      serviceAccountName: cluster-autoscaler
      containers:
      - image: k8s.gcr.io/autoscaling/cluster-autoscaler:v1.21.0
        name: cluster-autoscaler
        resources:
          limits:
            cpu: 100m
            memory: 300Mi
          requests:
            cpu: 100m
            memory: 300Mi
        command:
        - ./cluster-autoscaler
        - --v=4
        - --stderrthreshold=info
        - --cloud-provider=aws
        - --skip-nodes-with-local-storage=false
        - --expander=least-waste
        - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/teraffi-production
```

This comprehensive infrastructure and deployment guide provides everything needed to deploy TERAFFI's backend at production scale with high availability, security, and cost optimization.