---
title: Federation
description: Connect your Eltrix server to the Matrix network
nav_title: Federation
category: advanced
order: 7
---

Federation allows your Eltrix server to communicate with other Matrix homeservers, enabling users to join rooms and chat with users on different servers.

## How Federation Works

When a user on your server joins a room hosted on another server:

1. Your server discovers the remote server via `.well-known` or DNS SRV records
2. Servers exchange signing keys to verify authenticity
3. Events are sent between servers over HTTPS (port 8448)
4. Each server maintains its own copy of the room state

## Prerequisites

For federation to work, you need:

- A **public domain name** (e.g., `matrix.example.com`)
- **Valid TLS certificates** (Let's Encrypt works great)
- **Port 8448** accessible from the internet
- Proper **DNS configuration**

## DNS Configuration

### Option 1: Delegation via .well-known

If your Matrix server runs on a different host than your main domain:

```
# Your domain: example.com
# Matrix server: matrix.example.com
```

Create `https://example.com/.well-known/matrix/server`:

```json
{
  "m.server": "matrix.example.com:8448"
}
```

And `https://example.com/.well-known/matrix/client`:

```json
{
  "m.homeserver": {
    "base_url": "https://matrix.example.com"
  }
}
```

### Option 2: DNS SRV Records

Alternative to `.well-known`:

```dns
_matrix._tcp.example.com. 3600 IN SRV 10 0 8448 matrix.example.com.
```

### Option 3: Direct (Same Domain)

If your server name matches your Matrix server hostname:

```bash
export ELTRIX_SERVER_NAME=matrix.example.com
# Server runs at matrix.example.com:8448
# No delegation needed
```

## Configuration

### Enable Federation

Federation is enabled by default:

```bash
export ELTRIX_FEDERATION_ENABLED=true
```

### Server Name

Your server name is your identity in the Matrix network:

```bash
export ELTRIX_SERVER_NAME=example.com
```

This cannot be changed after users are created.

### Signing Keys

Eltrix automatically generates signing keys on first run. To backup or restore:

```bash
# Backup
docker compose exec eltrix ./bin/eltrix eval \
  'IO.puts(Eltrix.Federation.export_keys())'

# The keys are also stored in the database
```

## Access Control

### Allowlist Mode

Only federate with specific servers:

```bash
export ELTRIX_FEDERATION_WHITELIST=matrix.org,example.org,trusted.com
```

### Blocklist Mode

Block specific servers:

```bash
export ELTRIX_FEDERATION_BLACKLIST=spam-server.com,malicious.net
```

### Isolated Mode

Disable federation entirely (private server):

```bash
export ELTRIX_FEDERATION_ENABLED=false
```

## Verification

### Test Your Setup

Use the Matrix Federation Tester:

```bash
curl "https://federationtester.matrix.org/api/report?server_name=example.com"
```

Or visit: `https://federationtester.matrix.org/`

### Check from Eltrix

```bash
docker compose exec eltrix ./bin/eltrix eval \
  'Eltrix.Federation.test_outbound("matrix.org")'
```

### Common Issues

| Issue | Solution |
|-------|----------|
| Certificate errors | Ensure valid TLS cert on port 8448 |
| Connection refused | Check firewall allows port 8448 |
| Server not found | Verify DNS and .well-known |
| Key verification failed | Check server clocks are synced |

## Firewall Rules

Ensure these ports are accessible:

| Port | Protocol | Direction | Purpose |
|------|----------|-----------|---------|
| 8448 | TCP | Inbound | Federation API |
| 8448 | TCP | Outbound | Connecting to other servers |
| 443 | TCP | Outbound | .well-known lookups |

Example UFW rules:

```bash
ufw allow 8448/tcp
```

## Rate Limiting

Protect your server from federation abuse:

```bash
# Requests per second per remote server
export ELTRIX_FEDERATION_RATE_LIMIT=100

# Maximum concurrent connections per remote server
export ELTRIX_FEDERATION_MAX_CONNECTIONS=50
```

## Monitoring Federation

### Metrics

Eltrix exposes federation metrics:

- `eltrix_federation_requests_total` - Total requests sent/received
- `eltrix_federation_request_duration_seconds` - Request latency
- `eltrix_federation_errors_total` - Federation errors

### Logs

Enable debug logging for federation issues:

```bash
export ELTRIX_LOG_LEVEL=debug
export ELTRIX_LOG_FEDERATION=true
```

### Dashboard

View federation status:

```bash
docker compose exec eltrix ./bin/eltrix eval \
  'Eltrix.Federation.status() |> IO.inspect()'
```

## Security Considerations

### Key Rotation

Periodically rotate signing keys:

```bash
docker compose exec eltrix ./bin/eltrix eval \
  'Eltrix.Federation.rotate_keys()'
```

Old keys remain valid for verification until they expire.

### Server Verification

Eltrix verifies all incoming federation requests:

1. TLS certificate validation
2. Server key signature verification
3. Event signature verification
4. Authorization rules checking

### Spam Prevention

Recommended settings for public servers:

```bash
export ELTRIX_FEDERATION_RATE_LIMIT=50
export ELTRIX_MEDIA_QUARANTINE_ENABLED=true
export ELTRIX_INVITE_REQUIRE_KNOWN_SERVERS=true
```

## Troubleshooting

### Cannot Join Remote Rooms

1. Check federation tester results
2. Verify .well-known is accessible
3. Check port 8448 is open
4. Verify TLS certificates

```bash
# Test connectivity
openssl s_client -connect matrix.org:8448 -servername matrix.org
```

### Events Not Syncing

1. Check logs for federation errors
2. Verify both servers can reach each other
3. Check for key verification issues

```bash
docker compose logs eltrix | grep federation
```

### High Latency

1. Check network connectivity
2. Monitor database query times
3. Consider adding federation workers

```bash
export ELTRIX_FEDERATION_WORKERS=4
```

## Next Steps

- [Scaling Guide](/docs/scaling/) - Scale federation
- [API Reference](/docs/api/) - Federation API details
- [Configuration](/docs/configuration/) - All configuration options
