---
title: Scaling
description: Scale Eltrix for high availability
nav_title: Scaling
category: advanced
order: 6
---

Eltrix is built on Elixir and OTP, which provides excellent horizontal scaling capabilities. This guide covers scaling strategies for different deployment sizes.

## Architecture Overview

Eltrix uses a shared-nothing architecture where each node can handle any request:

```
                    ┌─────────────────┐
                    │  Load Balancer  │
                    └────────┬────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
    ┌────▼────┐        ┌────▼────┐        ┌────▼────┐
    │ Eltrix  │        │ Eltrix  │        │ Eltrix  │
    │ Node 1  │        │ Node 2  │        │ Node 3  │
    └────┬────┘        └────┬────┘        └────┬────┘
         │                   │                   │
         └───────────────────┼───────────────────┘
                             │
                    ┌────────▼────────┐
                    │   PostgreSQL    │
                    │    (Primary)    │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │    Replicas     │
                    └─────────────────┘
```

## Scaling Dimensions

### Vertical Scaling

For small to medium deployments, vertical scaling is often sufficient:

| Users | CPU | Memory | Pool Size |
|-------|-----|--------|-----------|
| < 100 | 2 | 2 GB | 10 |
| 100-500 | 4 | 4 GB | 20 |
| 500-1000 | 8 | 8 GB | 30 |
| 1000+ | Consider horizontal | | |

```bash
# Environment variables for vertical scaling
export ELTRIX_POOL_SIZE=20
export ELTRIX_WORKERS=8
```

### Horizontal Scaling

For larger deployments, scale horizontally by adding more Eltrix nodes:

```yaml
# Kubernetes deployment
spec:
  replicas: 5  # Add more replicas
```

## Database Scaling

### Connection Pooling

Use PgBouncer for connection pooling in large deployments:

```yaml
# docker-compose.yml
services:
  pgbouncer:
    image: edoburu/pgbouncer:latest
    environment:
      DATABASE_URL: postgres://postgres:password@db/eltrix
      POOL_MODE: transaction
      MAX_CLIENT_CONN: 1000
      DEFAULT_POOL_SIZE: 50
    ports:
      - "6432:6432"

  eltrix:
    environment:
      DATABASE_URL: postgres://postgres:password@pgbouncer:6432/eltrix
```

### Read Replicas

For read-heavy workloads, configure read replicas:

```bash
export DATABASE_URL=postgres://user:pass@primary/eltrix
export DATABASE_REPLICA_URL=postgres://user:pass@replica/eltrix
```

Eltrix will automatically route read queries to replicas.

## Caching

### Redis for Distributed Caching

Enable Redis for shared caching across nodes:

```bash
export ELTRIX_CACHE_BACKEND=redis
export REDIS_URL=redis://redis:6379
```

```yaml
# docker-compose.yml
services:
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
```

### Cache Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `ELTRIX_CACHE_BACKEND` | `local` | `local` or `redis` |
| `ELTRIX_CACHE_TTL` | `3600` | Default TTL in seconds |
| `REDIS_URL` | - | Redis connection URL |
| `REDIS_POOL_SIZE` | `10` | Redis connection pool size |

## Media Storage

For horizontal scaling, use S3-compatible storage:

```bash
export ELTRIX_MEDIA_STORE=s3
export ELTRIX_S3_BUCKET=eltrix-media
export ELTRIX_S3_REGION=us-east-1
export ELTRIX_S3_ACCESS_KEY=your-access-key
export ELTRIX_S3_SECRET_KEY=your-secret-key
```

This ensures all nodes can access uploaded media.

## Load Balancing

### Session Affinity

Eltrix doesn't require session affinity—any node can handle any request. However, for WebSocket connections (sync), sticky sessions can improve performance:

```yaml
# Kubernetes ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/affinity: "cookie"
    nginx.ingress.kubernetes.io/session-cookie-name: "eltrix-affinity"
    nginx.ingress.kubernetes.io/session-cookie-max-age: "3600"
```

### Health Check Endpoints

Configure your load balancer to use these endpoints:

| Endpoint | Use Case |
|----------|----------|
| `/health/live` | Liveness probe (is the process running?) |
| `/health/ready` | Readiness probe (can it serve traffic?) |
| `/health` | General health check |

## Distributed Erlang

For advanced use cases, Eltrix nodes can form an Erlang cluster for:

- Distributed presence tracking
- Cross-node event delivery
- Shared state

```bash
export ELTRIX_CLUSTER_ENABLED=true
export ELTRIX_CLUSTER_STRATEGY=kubernetes  # or dns, epmd
export ELTRIX_CLUSTER_KUBERNETES_SELECTOR=app=eltrix
```

## Monitoring

### Key Metrics

Monitor these metrics for scaling decisions:

| Metric | Threshold | Action |
|--------|-----------|--------|
| CPU utilization | > 70% | Add nodes |
| Memory usage | > 80% | Add memory or nodes |
| DB connections | > 80% pool | Increase pool or add replicas |
| Request latency | > 500ms p99 | Add nodes or optimize |
| Queue depth | Growing | Add workers |

### Prometheus Metrics

Eltrix exposes metrics at `/metrics`:

```bash
curl http://localhost:8008/metrics
```

Key metrics:

- `eltrix_http_requests_total`
- `eltrix_http_request_duration_seconds`
- `eltrix_sync_connections`
- `eltrix_federation_requests_total`
- `eltrix_db_query_duration_seconds`

## Example Configurations

### Small (< 500 users)

```yaml
# Single node
services:
  eltrix:
    image: ghcr.io/eltrix/eltrix:latest
    environment:
      ELTRIX_POOL_SIZE: 15
      ELTRIX_WORKERS: 4
    deploy:
      resources:
        limits:
          cpus: '4'
          memory: 4G
```

### Medium (500-5000 users)

```yaml
# 3 nodes with Redis
services:
  eltrix:
    image: ghcr.io/eltrix/eltrix:latest
    deploy:
      replicas: 3
    environment:
      ELTRIX_POOL_SIZE: 20
      ELTRIX_CACHE_BACKEND: redis
      REDIS_URL: redis://redis:6379
      ELTRIX_MEDIA_STORE: s3
```

### Large (5000+ users)

```yaml
# Kubernetes with HPA
spec:
  replicas: 5
  # HPA will scale 5-20 based on load

# External managed PostgreSQL
# External managed Redis cluster
# S3 for media
# PgBouncer for connection pooling
```

## Next Steps

- [Kubernetes Guide](/docs/kubernetes/) - Kubernetes-specific scaling
- [Federation Guide](/docs/federation/) - Scale federation
- [API Reference](/docs/api/) - API documentation
