# HTTP Proxy Setup

This guide covers how to configure and use HTTP proxies with Relyr for various networking scenarios.

## Overview

HTTP proxies act as intermediaries between your client applications and target servers. Relyr provides comprehensive proxy support with features like:

- HTTP/HTTPS proxy support
- SOCKS4/SOCKS5 proxy support
- Authentication mechanisms
- Proxy rotation and load balancing
- SSL/TLS termination

## Basic HTTP Proxy Configuration

### Environment Variables

The simplest way to configure proxy settings is through environment variables:

```bash
# HTTP proxy
export HTTP_PROXY=http://proxy.example.com:8080

# HTTPS proxy
export HTTPS_PROXY=https://proxy.example.com:8080

# No proxy for specific domains
export NO_PROXY=localhost,127.0.0.1,.example.com
```

### Configuration File

Create a `relyr.yml` configuration file:

```yaml
proxy:
  http:
    host: proxy.example.com
    port: 8080
    protocol: http
  https:
    host: proxy.example.com
    port: 8080
    protocol: https
  no_proxy:
    - localhost
    - 127.0.0.1
    - "*.example.com"
```

## Authentication

### Basic Authentication

For proxies requiring username/password authentication:

```yaml
proxy:
  http:
    host: proxy.example.com
    port: 8080
    auth:
      type: basic
      username: your_username
      password: your_password
```

Or via environment variables:

```bash
export HTTP_PROXY=http://username:password@proxy.example.com:8080
```

### Token-based Authentication

For modern authentication systems:

```yaml
proxy:
  http:
    host: proxy.example.com
    port: 8080
    auth:
      type: bearer
      token: your_api_token
```

## SOCKS Proxy Support

### SOCKS5 Configuration

```yaml
proxy:
  socks:
    host: socks.example.com
    port: 1080
    version: 5
    auth:
      username: your_username
      password: your_password
```

### SOCKS4 Configuration

```yaml
proxy:
  socks:
    host: socks.example.com
    port: 1080
    version: 4
```

## Advanced Proxy Features

### Proxy Rotation

Configure multiple proxies for load balancing and failover:

```yaml
proxy:
  rotation:
    enabled: true
    strategy: round_robin  # round_robin, random, failover
    proxies:
      - host: proxy1.example.com
        port: 8080
      - host: proxy2.example.com
        port: 8080
      - host: proxy3.example.com
        port: 8080
```

### Health Checks

Monitor proxy health and automatically failover:

```yaml
proxy:
  health_check:
    enabled: true
    interval: 30s
    timeout: 5s
    url: http://httpbin.org/ip
    expected_status: 200
```

### SSL/TLS Configuration

For HTTPS proxies with custom certificates:

```yaml
proxy:
  https:
    host: proxy.example.com
    port: 8080
    tls:
      verify_cert: true
      ca_cert_path: /path/to/ca.pem
      client_cert_path: /path/to/client.pem
      client_key_path: /path/to/client-key.pem
```

## Proxy Chains

Chain multiple proxies for enhanced privacy:

```yaml
proxy:
  chain:
    enabled: true
    proxies:
      - host: proxy1.example.com
        port: 8080
        type: http
      - host: proxy2.example.com
        port: 1080
        type: socks5
```

## Testing Proxy Configuration

### Command Line Testing

Test your proxy configuration:

```bash
# Test HTTP proxy
relyr test-proxy --type http --host proxy.example.com --port 8080

# Test with authentication
relyr test-proxy --type http --host proxy.example.com --port 8080 \
  --username user --password pass

# Test SOCKS proxy
relyr test-proxy --type socks5 --host socks.example.com --port 1080
```

### Programmatic Testing

```python
from relyr import ProxyClient

# Test proxy connectivity
client = ProxyClient(
    proxy_host="proxy.example.com",
    proxy_port=8080,
    proxy_type="http"
)

result = client.test_connection()
if result.success:
    print(f"Proxy working! IP: {result.external_ip}")
else:
    print(f"Proxy failed: {result.error}")
```

## Common Proxy Issues

### Connection Timeouts

If experiencing timeouts, adjust timeout settings:

```yaml
proxy:
  timeouts:
    connect: 10s
    read: 30s
    write: 30s
```

### DNS Resolution

Configure DNS resolution through proxy:

```yaml
proxy:
  dns:
    resolve_through_proxy: true
    servers:
      - 8.8.8.8
      - 1.1.1.1
```

### IPv6 Support

Enable IPv6 proxy support:

```yaml
proxy:
  ipv6:
    enabled: true
    prefer_ipv4: false
```

## Performance Optimization

### Connection Pooling

Optimize performance with connection pooling:

```yaml
proxy:
  connection_pool:
    max_connections: 100
    max_connections_per_host: 10
    keep_alive_timeout: 30s
```

### Compression

Enable compression for better performance:

```yaml
proxy:
  compression:
    enabled: true
    algorithms:
      - gzip
      - deflate
      - br
```

## Security Considerations

!!! warning "Security Best Practices"
    - Never log proxy credentials in plain text
    - Use encrypted connections (HTTPS/SOCKS5) when possible
    - Regularly rotate proxy credentials
    - Monitor proxy usage for suspicious activity

### Credential Management

Store credentials securely:

```yaml
proxy:
  credentials:
    source: env  # env, file, vault
    env_prefix: RELYR_PROXY_
```

## Monitoring and Logging

### Proxy Metrics

Monitor proxy performance:

```yaml
monitoring:
  proxy_metrics:
    enabled: true
    endpoint: /metrics
    include:
      - connection_count
      - response_time
      - error_rate
      - throughput
```

### Logging Configuration

Configure proxy-related logging:

```yaml
logging:
  proxy:
    level: info
    format: json
    include_headers: false
    sanitize_auth: true
```

## Examples

See the [Examples](../examples/basic.md) section for complete implementation examples and use cases.
