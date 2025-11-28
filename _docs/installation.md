---
title: Installation
description: Detailed installation instructions for Eltrix
nav_title: Installation
category: getting-started
order: 2
---

This guide covers all installation methods for Eltrix. Choose the approach that best fits your infrastructure.

## System Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| CPU | 2 cores | 4+ cores |
| RAM | 2 GB | 4+ GB |
| Storage | 20 GB SSD | 100+ GB SSD |
| PostgreSQL | 14 | 16 |

## Docker Installation

### Using Docker Compose (Recommended)

Create a `docker-compose.yml`:

```yaml
services:
  eltrix:
    image: ghcr.io/eltrix/eltrix:latest
    restart: unless-stopped
    environment:
      ELTRIX_SERVER_NAME: matrix.example.com
      ELTRIX_SECRET_KEY: ${SECRET_KEY}
      DATABASE_URL: postgres://postgres:postgres@db/eltrix
      ELTRIX_LOG_LEVEL: info
    ports:
      - "8008:8008"   # Client-Server API
      - "8448:8448"   # Federation API
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - eltrix_data:/var/lib/eltrix
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8008/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_DB: eltrix
      POSTGRES_PASSWORD: postgres
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

### Using Docker CLI

For simpler deployments:

```bash
# Create a network
docker network create eltrix-net

# Start PostgreSQL
docker run -d \
  --name eltrix-db \
  --network eltrix-net \
  -e POSTGRES_DB=eltrix \
  -e POSTGRES_PASSWORD=postgres \
  -v eltrix-postgres:/var/lib/postgresql/data \
  postgres:16-alpine

# Start Eltrix
docker run -d \
  --name eltrix \
  --network eltrix-net \
  -e ELTRIX_SERVER_NAME=matrix.example.com \
  -e ELTRIX_SECRET_KEY=$(openssl rand -base64 32) \
  -e DATABASE_URL=postgres://postgres:postgres@eltrix-db/eltrix \
  -p 8008:8008 \
  -p 8448:8448 \
  -v eltrix-data:/var/lib/eltrix \
  ghcr.io/eltrix/eltrix:latest
```

## Manual Installation

For production deployments or development, you may want to install Eltrix directly.

### Prerequisites

Install Elixir and Erlang:

```bash
# Ubuntu/Debian
sudo apt-get install elixir erlang-dev erlang-xmerl

# macOS
brew install elixir

# Using asdf (recommended)
asdf plugin add elixir
asdf plugin add erlang
asdf install erlang 26.2
asdf install elixir 1.16.0-otp-26
```

### Clone and Build

```bash
# Clone the repository
git clone https://github.com/eltrix/eltrix.git
cd eltrix

# Install dependencies
mix deps.get

# Compile
MIX_ENV=prod mix compile

# Build release
MIX_ENV=prod mix release
```

### Database Setup

```bash
# Create database
createdb eltrix

# Run migrations
MIX_ENV=prod mix ecto.migrate
```

### Start the Server

```bash
# Using the release
_build/prod/rel/eltrix/bin/eltrix start

# Or with mix (development)
MIX_ENV=prod mix phx.server
```

## Reverse Proxy Setup

For production, put Eltrix behind a reverse proxy like nginx or Caddy.

### Nginx Configuration

```nginx
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name matrix.example.com;

    ssl_certificate /etc/letsencrypt/live/matrix.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/matrix.example.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:8008;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}

# Federation port
server {
    listen 8448 ssl http2;
    listen [::]:8448 ssl http2;
    server_name matrix.example.com;

    ssl_certificate /etc/letsencrypt/live/matrix.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/matrix.example.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:8448;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Caddy Configuration

```caddyfile
matrix.example.com {
    reverse_proxy localhost:8008
}

matrix.example.com:8448 {
    reverse_proxy localhost:8448
}
```

## Next Steps

- [Configuration Guide](/docs/configuration/) - Customize your installation
- [Docker Guide](/docs/docker/) - Advanced Docker deployments
- [Kubernetes Guide](/docs/kubernetes/) - Deploy on Kubernetes
