---
title: API Reference
description: Eltrix API documentation
nav_title: API Reference
category: reference
order: 8
---

Eltrix implements the Matrix specification. This page covers Matrix APIs and Eltrix-specific endpoints.

## Matrix Specification

Eltrix implements the full Matrix Client-Server and Server-Server (Federation) APIs:

- **Matrix Spec Version**: 1.6
- **Client-Server API**: Full support
- **Server-Server API**: Full support
- **Application Service API**: Full support

For the complete Matrix API documentation, see [spec.matrix.org](https://spec.matrix.org/).

## Client-Server API

### Base URL

```
https://matrix.example.com/_matrix/client/v3/
```

### Authentication

Most endpoints require an access token:

```bash
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  https://matrix.example.com/_matrix/client/v3/account/whoami
```

### Common Endpoints

#### Get Server Versions

```http
GET /_matrix/client/versions
```

Response:

```json
{
  "versions": ["v1.1", "v1.2", "v1.3", "v1.4", "v1.5", "v1.6"],
  "unstable_features": {
    "org.matrix.msc3575": true
  }
}
```

#### Login

```http
POST /_matrix/client/v3/login
Content-Type: application/json

{
  "type": "m.login.password",
  "identifier": {
    "type": "m.id.user",
    "user": "username"
  },
  "password": "password"
}
```

Response:

```json
{
  "user_id": "@username:example.com",
  "access_token": "syt_...",
  "device_id": "DEVICE123"
}
```

#### Sync

```http
GET /_matrix/client/v3/sync?timeout=30000
Authorization: Bearer ACCESS_TOKEN
```

#### Send Message

```http
PUT /_matrix/client/v3/rooms/{roomId}/send/m.room.message/{txnId}
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json

{
  "msgtype": "m.text",
  "body": "Hello, Matrix!"
}
```

## Sliding Sync (MSC3575)

Eltrix supports Sliding Sync for efficient mobile synchronization:

```http
POST /_matrix/client/unstable/org.matrix.msc3575/sync
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json

{
  "lists": {
    "rooms": {
      "ranges": [[0, 20]],
      "sort": ["by_recency"],
      "required_state": [
        ["m.room.name", ""],
        ["m.room.avatar", ""]
      ],
      "timeline_limit": 10
    }
  }
}
```

## Federation API

### Base URL

```
https://matrix.example.com:8448/_matrix/federation/v1/
```

### Server Discovery

```http
GET /.well-known/matrix/server
```

Response:

```json
{
  "m.server": "matrix.example.com:8448"
}
```

### Get Server Keys

```http
GET /_matrix/key/v2/server
```

Response:

```json
{
  "server_name": "example.com",
  "verify_keys": {
    "ed25519:abc123": {
      "key": "base64+encoded+key"
    }
  },
  "old_verify_keys": {},
  "valid_until_ts": 1234567890000
}
```

## Eltrix Admin API

Eltrix provides additional admin endpoints.

### Authentication

Admin endpoints require an admin access token:

```bash
curl -H "Authorization: Bearer ADMIN_ACCESS_TOKEN" \
  https://matrix.example.com/_eltrix/admin/v1/users
```

### List Users

```http
GET /_eltrix/admin/v1/users?limit=100&offset=0
```

Response:

```json
{
  "users": [
    {
      "user_id": "@alice:example.com",
      "displayname": "Alice",
      "admin": false,
      "deactivated": false,
      "created_at": "2024-01-15T10:30:00Z"
    }
  ],
  "total": 150,
  "next_offset": 100
}
```

### Create User

```http
POST /_eltrix/admin/v1/users
Content-Type: application/json

{
  "username": "newuser",
  "password": "secure-password",
  "admin": false
}
```

### Deactivate User

```http
POST /_eltrix/admin/v1/users/@user:example.com/deactivate
```

### Server Statistics

```http
GET /_eltrix/admin/v1/statistics
```

Response:

```json
{
  "users": {
    "total": 1500,
    "active_30d": 1200
  },
  "rooms": {
    "total": 5000,
    "local": 500
  },
  "media": {
    "total_size_bytes": 10737418240,
    "count": 50000
  },
  "federation": {
    "known_servers": 1000,
    "active_servers": 500
  }
}
```

### Room Administration

```http
# List rooms
GET /_eltrix/admin/v1/rooms

# Get room details
GET /_eltrix/admin/v1/rooms/{roomId}

# Delete room
DELETE /_eltrix/admin/v1/rooms/{roomId}

# Purge room history
POST /_eltrix/admin/v1/rooms/{roomId}/purge
Content-Type: application/json

{
  "purge_up_to_ts": 1234567890000
}
```

## Health Endpoints

### Basic Health

```http
GET /health
```

Response:

```json
{
  "status": "healthy"
}
```

### Readiness

```http
GET /health/ready
```

Response (healthy):

```json
{
  "status": "ready",
  "checks": {
    "database": "ok",
    "cache": "ok"
  }
}
```

### Liveness

```http
GET /health/live
```

Response:

```json
{
  "status": "alive"
}
```

## Metrics

Prometheus-compatible metrics:

```http
GET /metrics
```

Response:

```
# HELP eltrix_http_requests_total Total HTTP requests
# TYPE eltrix_http_requests_total counter
eltrix_http_requests_total{method="GET",path="/sync",status="200"} 12345

# HELP eltrix_http_request_duration_seconds HTTP request duration
# TYPE eltrix_http_request_duration_seconds histogram
eltrix_http_request_duration_seconds_bucket{le="0.1"} 10000
```

## Rate Limiting

Eltrix implements rate limiting per the Matrix spec:

```http
HTTP/1.1 429 Too Many Requests
Content-Type: application/json

{
  "errcode": "M_LIMIT_EXCEEDED",
  "error": "Too many requests",
  "retry_after_ms": 5000
}
```

## Error Responses

All errors follow the Matrix error format:

```json
{
  "errcode": "M_NOT_FOUND",
  "error": "Room not found"
}
```

Common error codes:

| Code | Description |
|------|-------------|
| `M_FORBIDDEN` | Access denied |
| `M_UNKNOWN_TOKEN` | Invalid access token |
| `M_NOT_FOUND` | Resource not found |
| `M_LIMIT_EXCEEDED` | Rate limited |
| `M_BAD_JSON` | Invalid JSON |
| `M_UNKNOWN` | Unknown error |

## SDKs and Libraries

Use these libraries to interact with Eltrix:

| Language | Library |
|----------|---------|
| JavaScript | [matrix-js-sdk](https://github.com/matrix-org/matrix-js-sdk) |
| Python | [matrix-nio](https://github.com/poljar/matrix-nio) |
| Rust | [matrix-rust-sdk](https://github.com/matrix-org/matrix-rust-sdk) |
| Go | [mautrix-go](https://github.com/mautrix/go) |
| Elixir | [polyjuice_client](https://gitlab.com/polyjuice/polyjuice_client) |

## Next Steps

- [Configuration](/docs/configuration/) - Configure API behavior
- [Federation](/docs/federation/) - Federation API details
- [Getting Started](/docs/getting-started/) - Quick start guide
