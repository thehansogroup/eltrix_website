---
title: Docker
description: Deploy Eltrix with Docker
nav_title: Docker
category: deployment
order: 4
---

Docker is the recommended way to deploy Eltrix. This guide covers Docker-specific configuration and best practices.

## Official Images

Eltrix images are published to GitHub Container Registry:

```bash
# Latest stable release
ghcr.io/eltrix/eltrix:latest

# Specific version
ghcr.io/eltrix/eltrix:1.0.0

# Development builds
ghcr.io/eltrix/eltrix:main
```

## Docker Compose

### Basic Setup

```yaml
# docker-compose.yml
services:
  eltrix:
    image: ghcr.io/eltrix/eltrix:latest
    restart: unless-stopped
    environment:
      ELTRIX_SERVER_NAME: matrix.example.com
      ELTRIX_SECRET_KEY: ${SECRET_KEY}
      DATABASE_URL: postgres://postgres:${POSTGRES_PASSWORD}@db/eltrix
    ports:
      - "8008:8008"
      - "8448:8448"
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - eltrix_data:/var/lib/eltrix

  db:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_DB: eltrix
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  eltrix_data:
  postgres_data:
```

Create a `.env` file:

```bash
SECRET_KEY=your-secret-key-here
POSTGRES_PASSWORD=your-postgres-password
```

### Production Setup with Traefik

```yaml
# docker-compose.yml
services:
  traefik:
    image: traefik:v3.0
    command:
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--entrypoints.federation.address=:8448"
      - "--certificatesresolvers.letsencrypt.acme.httpchallenge=true"
      - "--certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web"
      - "--certificatesresolvers.letsencrypt.acme.email=admin@example.com"
      - "--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json"
    ports:
      - "80:80"
      - "443:443"
      - "8448:8448"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - letsencrypt:/letsencrypt

  eltrix:
    image: ghcr.io/eltrix/eltrix:latest
    restart: unless-stopped
    environment:
      ELTRIX_SERVER_NAME: matrix.example.com
      ELTRIX_SECRET_KEY: ${SECRET_KEY}
      DATABASE_URL: postgres://postgres:${POSTGRES_PASSWORD}@db/eltrix
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - eltrix_data:/var/lib/eltrix
    labels:
      - "traefik.enable=true"
      # Client-Server API
      - "traefik.http.routers.eltrix.rule=Host(`matrix.example.com`)"
      - "traefik.http.routers.eltrix.entrypoints=websecure"
      - "traefik.http.routers.eltrix.tls.certresolver=letsencrypt"
      - "traefik.http.services.eltrix.loadbalancer.server.port=8008"
      # Federation API
      - "traefik.http.routers.eltrix-fed.rule=Host(`matrix.example.com`)"
      - "traefik.http.routers.eltrix-fed.entrypoints=federation"
      - "traefik.http.routers.eltrix-fed.tls.certresolver=letsencrypt"
      - "traefik.http.services.eltrix-fed.loadbalancer.server.port=8448"

  db:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_DB: eltrix
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  letsencrypt:
  eltrix_data:
  postgres_data:
```

## Health Checks

The Eltrix container exposes health endpoints:

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:8008/health"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 10s
```

Available endpoints:

| Endpoint | Description |
|----------|-------------|
| `/health` | Basic health check |
| `/health/ready` | Readiness check (DB connected) |
| `/health/live` | Liveness check |

## Resource Limits

Set appropriate resource limits for production:

```yaml
services:
  eltrix:
    image: ghcr.io/eltrix/eltrix:latest
    deploy:
      resources:
        limits:
          cpus: '4'
          memory: 4G
        reservations:
          cpus: '2'
          memory: 2G
```

## Logging

Configure Docker logging for production:

```yaml
services:
  eltrix:
    image: ghcr.io/eltrix/eltrix:latest
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "5"
    environment:
      ELTRIX_LOG_FORMAT: json
```

## Volume Mounts

### Data Persistence

```yaml
volumes:
  # Named volumes (recommended)
  eltrix_data:
    driver: local

  # Or bind mounts for easier backup
  eltrix_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /srv/eltrix/data
```

### Backup Strategy

```bash
# Stop the container
docker compose stop eltrix

# Backup database
docker compose exec db pg_dump -U postgres eltrix > backup.sql

# Backup media files
docker run --rm -v eltrix_data:/data -v $(pwd):/backup \
  alpine tar czf /backup/eltrix-data.tar.gz /data

# Restart
docker compose start eltrix
```

## Upgrading

```bash
# Pull new image
docker compose pull eltrix

# Recreate container
docker compose up -d eltrix

# Check logs
docker compose logs -f eltrix
```

## Administration Commands

Run admin commands inside the container:

```bash
# Create a user
docker compose exec eltrix ./bin/eltrix eval \
  'Eltrix.Admin.create_user("username", "password")'

# Create an admin user
docker compose exec eltrix ./bin/eltrix eval \
  'Eltrix.Admin.create_user("admin", "password", admin: true)'

# Reset a password
docker compose exec eltrix ./bin/eltrix eval \
  'Eltrix.Admin.reset_password("username", "new-password")'

# Generate a registration token
docker compose exec eltrix ./bin/eltrix eval \
  'Eltrix.Admin.create_registration_token()'
```

## Next Steps

- [Kubernetes Guide](/docs/kubernetes/) - Deploy on Kubernetes
- [Scaling Guide](/docs/scaling/) - Scale your deployment
- [Configuration](/docs/configuration/) - All configuration options
