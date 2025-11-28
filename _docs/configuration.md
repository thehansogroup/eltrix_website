---
title: Configuration
description: Configure Eltrix for your environment
nav_title: Configuration
category: getting-started
order: 3
---

Eltrix is configured primarily through environment variables. This keeps configuration simple and container-friendly.

## Essential Configuration

These variables must be set for Eltrix to run:

| Variable | Description | Example |
|----------|-------------|---------|
| `ELTRIX_SERVER_NAME` | Your Matrix server name (domain) | `matrix.example.com` |
| `ELTRIX_SECRET_KEY` | Secret key for signing tokens | `$(openssl rand -base64 32)` |
| `DATABASE_URL` | PostgreSQL connection string | `postgres://user:pass@host/db` |

```bash
export ELTRIX_SERVER_NAME=matrix.example.com
export ELTRIX_SECRET_KEY=$(openssl rand -base64 32)
export DATABASE_URL=postgres://postgres:postgres@localhost/eltrix
```

## Server Configuration

### Network Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `ELTRIX_PORT` | `8008` | Client-Server API port |
| `ELTRIX_FEDERATION_PORT` | `8448` | Server-Server (federation) port |
| `ELTRIX_BIND_ADDRESS` | `0.0.0.0` | Address to bind to |

### Performance Tuning

| Variable | Default | Description |
|----------|---------|-------------|
| `ELTRIX_POOL_SIZE` | `10` | Database connection pool size |
| `ELTRIX_MAX_UPLOAD_SIZE` | `50` | Max upload size in MB |
| `ELTRIX_WORKERS` | `auto` | Number of worker processes |

```bash
# High-traffic configuration
export ELTRIX_POOL_SIZE=25
export ELTRIX_MAX_UPLOAD_SIZE=100
export ELTRIX_WORKERS=8
```

## Registration Settings

Control how users can register on your server:

| Variable | Default | Description |
|----------|---------|-------------|
| `ELTRIX_REGISTRATION_ENABLED` | `false` | Allow public registration |
| `ELTRIX_REGISTRATION_REQUIRES_TOKEN` | `false` | Require registration token |
| `ELTRIX_REGISTRATION_TOKEN` | - | Token for registration |

```bash
# Enable registration with a token
export ELTRIX_REGISTRATION_ENABLED=true
export ELTRIX_REGISTRATION_REQUIRES_TOKEN=true
export ELTRIX_REGISTRATION_TOKEN=my-secret-token
```

## Federation Settings

Configure how your server interacts with the Matrix network:

| Variable | Default | Description |
|----------|---------|-------------|
| `ELTRIX_FEDERATION_ENABLED` | `true` | Enable federation |
| `ELTRIX_FEDERATION_WHITELIST` | - | Comma-separated allowed servers |
| `ELTRIX_FEDERATION_BLACKLIST` | - | Comma-separated blocked servers |

```bash
# Restrict federation to specific servers
export ELTRIX_FEDERATION_WHITELIST=matrix.org,example.com
```

## Authentication

### Local Authentication

| Variable | Default | Description |
|----------|---------|-------------|
| `ELTRIX_PASSWORD_MIN_LENGTH` | `8` | Minimum password length |
| `ELTRIX_PASSWORD_REQUIRE_SPECIAL` | `false` | Require special characters |
| `ELTRIX_SESSION_LIFETIME` | `604800` | Session lifetime in seconds (default: 7 days) |

### OpenID Connect (OIDC)

Eltrix supports OIDC for single sign-on:

| Variable | Description |
|----------|-------------|
| `ELTRIX_OIDC_ENABLED` | Enable OIDC authentication |
| `ELTRIX_OIDC_ISSUER` | OIDC provider URL |
| `ELTRIX_OIDC_CLIENT_ID` | Client ID |
| `ELTRIX_OIDC_CLIENT_SECRET` | Client secret |
| `ELTRIX_OIDC_SCOPES` | Requested scopes (default: `openid profile email`) |

```bash
# Keycloak example
export ELTRIX_OIDC_ENABLED=true
export ELTRIX_OIDC_ISSUER=https://keycloak.example.com/realms/matrix
export ELTRIX_OIDC_CLIENT_ID=eltrix
export ELTRIX_OIDC_CLIENT_SECRET=your-client-secret
```

## Media Storage

Configure where uploaded files are stored:

| Variable | Default | Description |
|----------|---------|-------------|
| `ELTRIX_MEDIA_STORE` | `local` | Storage backend: `local` or `s3` |
| `ELTRIX_MEDIA_PATH` | `/var/lib/eltrix/media` | Local storage path |

### S3-Compatible Storage

```bash
export ELTRIX_MEDIA_STORE=s3
export ELTRIX_S3_BUCKET=eltrix-media
export ELTRIX_S3_REGION=us-east-1
export ELTRIX_S3_ACCESS_KEY=your-access-key
export ELTRIX_S3_SECRET_KEY=your-secret-key
export ELTRIX_S3_ENDPOINT=https://s3.amazonaws.com  # Optional, for MinIO etc.
```

## Logging

| Variable | Default | Description |
|----------|---------|-------------|
| `ELTRIX_LOG_LEVEL` | `info` | Log level: `debug`, `info`, `warn`, `error` |
| `ELTRIX_LOG_FORMAT` | `text` | Log format: `text` or `json` |

```bash
# Production logging
export ELTRIX_LOG_LEVEL=info
export ELTRIX_LOG_FORMAT=json
```

## Example Configurations

### Development

```bash
export ELTRIX_SERVER_NAME=localhost
export ELTRIX_SECRET_KEY=dev-secret-key
export DATABASE_URL=postgres://postgres:postgres@localhost/eltrix_dev
export ELTRIX_REGISTRATION_ENABLED=true
export ELTRIX_LOG_LEVEL=debug
```

### Production (Small)

```bash
export ELTRIX_SERVER_NAME=matrix.example.com
export ELTRIX_SECRET_KEY=$(openssl rand -base64 32)
export DATABASE_URL=postgres://eltrix:password@localhost/eltrix
export ELTRIX_POOL_SIZE=10
export ELTRIX_REGISTRATION_ENABLED=false
export ELTRIX_LOG_LEVEL=info
export ELTRIX_LOG_FORMAT=json
```

### Production (Large Scale)

```bash
export ELTRIX_SERVER_NAME=matrix.example.com
export ELTRIX_SECRET_KEY=$(openssl rand -base64 32)
export DATABASE_URL=postgres://eltrix:password@db-cluster/eltrix
export ELTRIX_POOL_SIZE=50
export ELTRIX_WORKERS=16
export ELTRIX_MEDIA_STORE=s3
export ELTRIX_S3_BUCKET=eltrix-media
export ELTRIX_S3_REGION=us-east-1
export ELTRIX_LOG_LEVEL=info
export ELTRIX_LOG_FORMAT=json
```

## Next Steps

- [Docker Guide](/docs/docker/) - Container-specific configuration
- [Kubernetes Guide](/docs/kubernetes/) - Kubernetes deployments
- [Scaling Guide](/docs/scaling/) - Scale your deployment
