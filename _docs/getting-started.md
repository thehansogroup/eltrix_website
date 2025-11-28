---
title: Getting Started
description: Get Eltrix up and running in minutes
nav_title: Getting Started
category: getting-started
order: 1
---

Welcome to Eltrix, the Elixir-powered Matrix homeserver. This guide will help you get a basic instance running quickly.

## Prerequisites

Before you begin, ensure you have:

- **Docker** and **Docker Compose** installed (recommended)
- OR **Elixir 1.15+** and **PostgreSQL 14+** for manual installation
- A domain name with DNS configured (for federation)
- Basic familiarity with command-line tools

## Quick Start with Docker

The fastest way to get started is with Docker Compose:

```yaml
# docker-compose.yml
services:
  eltrix:
    image: ghcr.io/eltrix/eltrix:latest
    environment:
      ELTRIX_SERVER_NAME: matrix.example.com
      ELTRIX_SECRET_KEY: ${SECRET_KEY}
      DATABASE_URL: postgres://postgres:postgres@db/eltrix
    ports:
      - "8008:8008"
      - "8448:8448"
    depends_on:
      - db
    volumes:
      - eltrix_data:/var/lib/eltrix

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: eltrix
      POSTGRES_PASSWORD: postgres
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  eltrix_data:
  postgres_data:
```

Generate a secret key and start the services:

```bash
# Generate a secure secret key
export SECRET_KEY=$(openssl rand -base64 32)

# Start Eltrix
docker compose up -d
```

## Verify Installation

Check that Eltrix is running:

```bash
curl http://localhost:8008/_matrix/client/versions
```

You should see a JSON response listing supported Matrix spec versions:

```json
{
  "versions": ["v1.1", "v1.2", "v1.3", "v1.4", "v1.5", "v1.6"]
}
```

## Create Your First User

Create an admin user to get started:

```bash
docker compose exec eltrix ./bin/eltrix eval \
  'Eltrix.Admin.create_user("admin", "your-password", admin: true)'
```

## Connect a Client

You can now connect with any Matrix client:

1. Open [Element Web](https://app.element.io) or your preferred Matrix client
2. Click "Sign in"
3. Enter your homeserver URL: `https://matrix.example.com`
4. Sign in with the credentials you created

## Next Steps

- [Installation Guide](/docs/installation/) - Detailed installation options
- [Configuration](/docs/configuration/) - Configure Eltrix for your needs
- [Federation](/docs/federation/) - Connect with the Matrix network
