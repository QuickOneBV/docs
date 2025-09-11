# Quick Start Guide

Get up and running with Relyr in just a few minutes! This guide will walk you through the basics of setting up and using Relyr as your proxy solution.

## Prerequisites

Before you begin, ensure you have:

- Relyr installed (see [Installation Guide](installation.md))
- Basic understanding of HTTP proxies
- Terminal/command line access

## 5-Minute Setup

### Step 1: Initialize Configuration

```bash
# Create configuration directory
mkdir -p ~/.config/relyr

# Generate default configuration
relyr config init
```

This creates a basic configuration file at `~/.config/relyr/relyr.yml`:

```yaml
server:
  host: 0.0.0.0
  port: 8080
  
logging:
  level: info
  
proxy:
  enabled: true
```

### Step 2: Start Relyr

```bash
# Start Relyr server
relyr server

# Or run in background
relyr server --daemon
```

You should see output similar to:

```
[INFO] Starting Relyr v1.0.0
[INFO] Server listening on 0.0.0.0:8080
[INFO] Proxy server ready
```

### Step 3: Test Your Proxy

Open a new terminal and test the proxy:

```bash
# Test with curl
curl -x http://localhost:8080 http://httpbin.org/ip

# Expected output:
# {
#   "origin": "YOUR_PROXY_IP"
# }
```

Congratulations! 🎉 You now have a working Relyr proxy server.

## Basic Usage Examples

### HTTP Requests Through Proxy

=== "curl"

    ```bash
    # Basic HTTP request
    curl -x http://localhost:8080 https://api.github.com/user
    
    # With authentication
    curl -x http://username:password@localhost:8080 https://api.example.com
    
    # HTTPS request
    curl -x http://localhost:8080 https://httpbin.org/get
    ```

=== "Python"

    ```python
    import requests
    
    # Configure proxy
    proxies = {
        'http': 'http://localhost:8080',
        'https': 'http://localhost:8080'
    }
    
    # Make request through proxy
    response = requests.get('https://httpbin.org/ip', proxies=proxies)
    print(response.json())
    ```

=== "Node.js"

    ```javascript
    const axios = require('axios');
    
    // Configure proxy
    const config = {
        proxy: {
            host: 'localhost',
            port: 8080
        }
    };
    
    // Make request through proxy
    axios.get('https://httpbin.org/ip', config)
        .then(response => console.log(response.data))
        .catch(error => console.error(error));
    ```

=== "Browser"

    Configure your browser to use `localhost:8080` as HTTP proxy:

    **Chrome/Chromium:**
    ```bash
    google-chrome --proxy-server="http://localhost:8080"
    ```

    **Firefox:**
    1. Go to Settings → Network Settings
    2. Select "Manual proxy configuration"
    3. HTTP Proxy: `localhost`, Port: `8080`
    4. Check "Use this proxy server for all protocols"

## Common Configuration

### Custom Port

```yaml
server:
  port: 3128  # Use port 3128 instead of 8080
```

### Enable Authentication

```yaml
auth:
  enabled: true
  method: basic
  users:
    - username: admin
      password: secretpassword
    - username: user
      password: userpass
```

Test with authentication:

```bash
curl -x http://admin:secretpassword@localhost:8080 https://httpbin.org/ip
```

### Logging Configuration

```yaml
logging:
  level: debug
  format: json
  output: /var/log/relyr/relyr.log
```

### Upstream Proxy

Chain proxies by configuring an upstream proxy:

```yaml
upstream:
  enabled: true
  proxy: http://upstream.example.com:8080
  auth:
    username: upstream_user
    password: upstream_pass
```

## Environment Variables

You can also configure Relyr using environment variables:

```bash
# Server configuration
export RELYR_SERVER_HOST=0.0.0.0
export RELYR_SERVER_PORT=8080

# Logging
export RELYR_LOG_LEVEL=info

# Authentication
export RELYR_AUTH_ENABLED=true
export RELYR_AUTH_USERNAME=admin
export RELYR_AUTH_PASSWORD=secret

# Start server with environment config
relyr server
```

## Health Checks

Monitor your Relyr instance:

```bash
# Check server health
curl http://localhost:8080/health

# Get server statistics
curl http://localhost:8080/stats

# Get configuration info
relyr config show
```

## Docker Quick Start

For containerized deployment:

```bash
# Run Relyr in Docker
docker run -d \
  --name relyr \
  -p 8080:8080 \
  relyr/relyr:latest

# Test the containerized proxy
curl -x http://localhost:8080 https://httpbin.org/ip
```

## Performance Testing

Test your proxy performance:

```bash
# Simple performance test
time curl -x http://localhost:8080 https://httpbin.org/get

# Load testing with Apache Bench
ab -n 1000 -c 10 -X localhost:8080 https://httpbin.org/get

# Using wrk for more advanced testing
wrk -t4 -c100 -d30s --script proxy.lua https://httpbin.org/get
```

## Next Steps

Now that you have Relyr running, explore these advanced features:

### 🔧 Configuration
- [HTTP Proxy Setup](proxy-setup.md) - Advanced proxy configurations
- [Environment Variables](environment.md) - Complete environment variable reference
- [Advanced Configuration](advanced-config.md) - Performance tuning and advanced features

### 📚 API Integration
- [API Overview](api/overview.md) - REST API documentation
- [Authentication](api/authentication.md) - API authentication methods
- [Endpoints](api/endpoints.md) - Complete API reference

### 💡 Examples
- [Basic Usage](examples/basic.md) - More usage examples
- [Advanced Examples](examples/advanced.md) - Complex configurations
- [Integration Patterns](examples/integration.md) - Integration with other tools

### 🛠️ Troubleshooting
- [Common Issues](troubleshooting/common-issues.md) - Solutions to common problems
- [FAQ](troubleshooting/faq.md) - Frequently asked questions
- [Performance Tips](troubleshooting/performance.md) - Optimization guide

## Getting Help

If you run into issues:

1. Check the logs: `tail -f ~/.config/relyr/logs/relyr.log`
2. Validate your configuration: `relyr config validate`
3. Review our [troubleshooting guide](troubleshooting/common-issues.md)
4. Join our [community forum](https://community.relyr.io)
5. Report bugs on [GitHub](https://github.com/relyr/relyr/issues)

## Configuration File Reference

Here's a complete basic configuration file for reference:

```yaml
# ~/.config/relyr/relyr.yml

# Server configuration
server:
  host: 0.0.0.0
  port: 8080
  read_timeout: 30s
  write_timeout: 30s

# Logging configuration
logging:
  level: info
  format: text
  output: stdout

# Proxy configuration
proxy:
  enabled: true
  protocols:
    - http
    - https
  
# Authentication (optional)
auth:
  enabled: false
  method: basic
  users: []

# Monitoring (optional)
monitoring:
  enabled: true
  metrics_port: 9090
  health_check_port: 8081

# Performance tuning
performance:
  max_connections: 1000
  connection_timeout: 10s
  keep_alive_timeout: 60s
```

That's it! You're now ready to use Relyr as your proxy solution. 🚀
