# TERAFFI GraphRAG Services - Docker Deployment Guide

This guide provides comprehensive instructions for deploying the TERAFFI GraphRAG services using Docker containers.

## Overview

The TERAFFI GraphRAG system consists of 11 containerized services that work together to provide intelligent data crawling, entity resolution, and graph-based retrieval:

### Core Services
- **LLM Gateway** (Port 3007): AI model routing and fallback management
- **GraphRAG Service** (Port 3008): Hybrid retrieval combining vector and graph search
- **Entity Resolution** (Port 3010): Intelligent entity deduplication and merging
- **Data Crawler** (Port 3012): Automated data ingestion from multiple sources
- **Scheduler** (Port 3011): Job orchestration and automation
- **Embedding Service** (Port 3009): Vector embedding generation

### Infrastructure Services
- **PostgreSQL** (Port 5433): Vector storage with pgvector extension
- **Neo4j** (Port 7474/7687): Graph database for relationships
- **Redis** (Port 6380): Caching and job queuing
- **Ollama** (Port 11434): Local LLM inference
- **Prometheus** (Port 9091): Monitoring and metrics

## Quick Start

### Prerequisites
- Docker >= 20.10
- Docker Compose >= 2.0
- 16GB+ RAM recommended
- 50GB+ available disk space

### 1. Environment Setup
```bash
# Copy environment template
cp .env.graphrag.example .env.graphrag

# Edit with your API keys and configuration
nano .env.graphrag
```

### 2. Quick Deployment
```bash
# Using Makefile (recommended)
make quick-start

# Or manual commands
docker-compose -f docker-compose.graphrag.yml build
docker-compose -f docker-compose.graphrag.yml up -d
```

### 3. Initialize Databases
```bash
make setup-db
```

### 4. Verify Deployment
```bash
make health
```

## Detailed Configuration

### Service Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Data Crawler  │    │  LLM Gateway    │    │ Entity Resolution│
│     :3012       │    │     :3007       │    │     :3010       │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          └──────────┬───────────┴───────────┬──────────┘
                     │                       │
           ┌─────────▼───────┐    ┌─────────▼───────┐
           │  GraphRAG       │    │   Scheduler     │
           │   Service       │    │    :3011        │
           │    :3008        │    └─────────────────┘
           └─────────┬───────┘
                     │
    ┌────────────────┼────────────────┐
    │                │                │
┌───▼────┐    ┌─────▼──────┐    ┌───▼────┐
│PostgreSQL│    │   Neo4j    │    │ Redis  │
│  :5433   │    │ :7474/7687 │    │ :6380  │
└──────────┘    └────────────┘    └────────┘
```

### Environment Variables

Key configuration options in `.env.graphrag`:

#### AI/LLM Configuration
- `OPENAI_API_KEY`: OpenAI API key for fallback
- `USE_LOCAL_LLM=true`: Prefer local Ollama models
- `MODEL_PREFERENCE=llama3.1:8b-instruct`: Default local model

#### Database Configuration
- `DATABASE_URL`: PostgreSQL connection string
- `NEO4J_URI`: Neo4j Bolt protocol URI
- `REDIS_URL`: Redis connection string

#### Service Thresholds
- `NAME_SIMILARITY_THRESHOLD=0.8`: Entity name matching threshold
- `EMBEDDING_SIMILARITY_THRESHOLD=0.85`: Vector similarity threshold
- `MERGE_CONFIDENCE_THRESHOLD=0.75`: Entity merge confidence threshold

## Development vs Production

### Development Environment
```bash
# Start with development overrides
make dev-up

# View development logs
make dev-logs

# Stop development environment
make dev-down
```

Development features:
- Hot reload for code changes
- Debug logging enabled
- Exposed database ports
- Development credentials

### Production Environment
```bash
# Production deployment
make deploy

# Rolling updates
make update

# Scale services
make scale SERVICE=graphrag-service REPLICAS=3
```

Production features:
- Optimized Docker images
- Health checks and restart policies
- Resource limits
- Monitoring integration

## Monitoring and Maintenance

### Health Monitoring
```bash
# Check all service health
make health

# View service status
make status

# Monitor resource usage
make stats
```

### Log Management
```bash
# View all logs
make logs

# View specific service logs
make logs-service SERVICE=llm-gateway

# View last 100 lines
make quick-logs
```

### Backup and Recovery
```bash
# Create data backup
make backup-data

# Restore from backup (manual process)
# Restore PostgreSQL: psql -U user -d db < backup.sql
# Restore Neo4j: Use Neo4j Browser to import
```

## Service-Specific Configuration

### LLM Gateway
- Routes requests between local Ollama and OpenAI
- Handles model fallback and load balancing
- Monitors model performance and availability

### Data Crawler
- Supports Instagram, Twitter, TikTok, and news sources
- Rate limiting and robots.txt compliance
- Configurable crawl schedules via Scheduler

### Entity Resolution
- Uses fuzzy matching, embeddings, and LLM decisions
- Automatic entity deduplication and merging
- Maintains relationship integrity during merges

### Scheduler
- Cron-based job scheduling
- Dependency management between jobs
- Health monitoring and failure recovery

## Troubleshooting

### Common Issues

#### Service Won't Start
```bash
# Check logs for specific service
docker logs teraffi-llm-gateway

# Check Docker daemon
systemctl status docker

# Check available resources
docker system df
```

#### Database Connection Issues
```bash
# Test PostgreSQL connection
docker exec -it teraffi-postgres-graphrag psql -U teraffi_user -d teraffi

# Test Neo4j connection
docker exec -it teraffi-neo4j cypher-shell -u neo4j -p teraffi_neo4j_password

# Test Redis connection
docker exec -it teraffi-redis-graphrag redis-cli ping
```

#### Performance Issues
```bash
# Monitor resource usage
make stats

# Check container health
make health

# Scale problematic services
make scale SERVICE=graphrag-service REPLICAS=2
```

### Debug Mode
```bash
# Start with debug logging
docker-compose -f docker-compose.graphrag.yml -f docker-compose.graphrag.override.yml up -d

# Access service shell
make shell-gateway
make shell-graphrag
```

## Security Considerations

### Production Security Checklist
- [ ] Change default passwords in `.env.graphrag`
- [ ] Use Docker secrets for sensitive data
- [ ] Enable TLS for external connections
- [ ] Configure firewall rules
- [ ] Regular security updates
- [ ] Monitor access logs

### Network Security
```bash
# Services communicate on isolated Docker network
# Only necessary ports are exposed to host
# Internal communication uses service names
```

## Performance Optimization

### Resource Allocation
- **LLM Gateway**: 2GB RAM minimum
- **GraphRAG Service**: 4GB RAM minimum  
- **Entity Resolution**: 3GB RAM minimum
- **PostgreSQL**: 4GB RAM minimum
- **Neo4j**: 6GB RAM minimum

### Scaling Strategies
```bash
# Horizontal scaling for compute services
make scale SERVICE=llm-gateway REPLICAS=3

# Vertical scaling via Docker Compose limits
# Add to service configuration:
# deploy:
#   resources:
#     limits:
#       cpus: '2.0'
#       memory: 4G
```

## API Endpoints

### Service Health Checks
- http://localhost:3007/health - LLM Gateway
- http://localhost:3008/health - GraphRAG Service  
- http://localhost:3009/health - Embedding Service
- http://localhost:3010/health - Entity Resolution
- http://localhost:3011/health - Scheduler
- http://localhost:3012/health - Data Crawler

### Monitoring
- http://localhost:9091 - Prometheus metrics
- http://localhost:7474 - Neo4j Browser

## Support

### Getting Help
1. Check service logs: `make logs-service SERVICE=<name>`
2. Verify health endpoints: `make health`
3. Review Docker system status: `docker system df`
4. Check resource usage: `make stats`

### Contributing
- Follow Docker best practices
- Include health checks in new services
- Update this documentation for changes
- Test both development and production modes

## Files Created

This deployment includes the following files:

### Docker Configuration
- `/docker-compose.graphrag.yml` - Main production configuration
- `/docker-compose.graphrag.override.yml` - Development overrides
- `/docker/Dockerfile.llm-gateway` - LLM Gateway container
- `/docker/Dockerfile.graphrag` - GraphRAG service container
- `/docker/Dockerfile.entity-resolution` - Entity Resolution container
- `/docker/Dockerfile.scheduler` - Scheduler service container
- `/docker/Dockerfile.crawler` - Data Crawler container
- `/docker/Dockerfile.embeddings` - Embedding service container

### Configuration Files
- `.env.graphrag.example` - Environment template
- `requirements.txt` - Python dependencies for embedding service
- `Makefile.graphrag` - Deployment automation

### Monitoring
- `/monitoring/prometheus-graphrag.yml` - Prometheus configuration
- `/monitoring/alert_rules.yml` - Alert rules for production monitoring

All services are production-ready with:
- ✅ Health checks
- ✅ Restart policies  
- ✅ Resource monitoring
- ✅ Proper networking
- ✅ Volume management
- ✅ Security best practices