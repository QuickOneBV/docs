# Frequently Asked Questions

Common questions and answers about Relyr.

## General Questions

### What is Relyr?

Relyr is a high-performance proxy server and networking solution designed for reliability, scalability, and ease of use. It supports HTTP/HTTPS proxies, SOCKS proxies, and advanced features like proxy rotation, authentication, and monitoring.

### What protocols does Relyr support?

Relyr supports:
- HTTP/1.1 and HTTP/2
- HTTPS with SSL/TLS termination
- SOCKS4 and SOCKS5
- WebSocket proxying
- TCP/UDP forwarding

### Is Relyr open source?

Yes, Relyr is open source and available on [GitHub](https://github.com/relyr/relyr) under the MIT license.

### What platforms does Relyr support?

Relyr runs on:
- Linux (all major distributions)
- macOS (Intel and Apple Silicon)
- Windows 10/11
- Docker containers
- Kubernetes clusters

## Installation & Setup

### How do I install Relyr?

See our [Installation Guide](../installation.md) for detailed instructions. The quickest method is:

```bash
# Using package managers
pip install relyr          # Python
npm install -g relyr       # Node.js
brew install relyr         # macOS
```

### What are the system requirements?

**Minimum:**
- 512MB RAM
- 100MB disk space
- Network connectivity

**Recommended:**
- 2GB+ RAM
- 1GB+ disk space
- 2+ CPU cores

### How do I configure Relyr?

Create a configuration file at `~/.config/relyr/relyr.yml`:

```yaml
server:
  host: 0.0.0.0
  port: 8080

logging:
  level: info

proxy:
  enabled: true
```

See the [Configuration Guide](../proxy-setup.md) for more details.

## Configuration Questions

### Can I use multiple upstream proxies?

Yes, Relyr supports proxy rotation and load balancing:

```yaml
proxy:
  rotation:
    enabled: true
    strategy: round_robin
    proxies:
      - host: proxy1.example.com
        port: 8080
      - host: proxy2.example.com
        port: 8080
```

### How do I enable authentication?

Configure authentication in your `relyr.yml`:

```yaml
auth:
  enabled: true
  method: basic
  users:
    - username: admin
      password: secretpassword
```

### Can I use custom SSL certificates?

Yes, configure SSL/TLS settings:

```yaml
server:
  tls:
    enabled: true
    cert_file: /path/to/cert.pem
    key_file: /path/to/key.pem
```

### How do I configure logging?

Customize logging in your configuration:

```yaml
logging:
  level: info          # debug, info, warn, error
  format: json         # json, text
  output: file         # stdout, file
  file: /var/log/relyr/relyr.log
```

## Performance Questions

### How many concurrent connections can Relyr handle?

This depends on your system resources. Relyr can handle thousands of concurrent connections. Configure limits:

```yaml
performance:
  max_connections: 10000
  max_connections_per_host: 100
```

### How do I optimize performance?

1. **Enable connection pooling:**
   ```yaml
   performance:
     connection_pooling:
       enabled: true
       max_idle_connections: 100
   ```

2. **Adjust timeouts:**
   ```yaml
   proxy:
     timeouts:
       connect: 5s
       read: 30s
   ```

3. **Use compression:**
   ```yaml
   proxy:
     compression:
       enabled: true
   ```

### What's the typical latency overhead?

Relyr adds minimal latency (typically 1-5ms) depending on:
- Network conditions
- Target server response time
- Configuration complexity
- System resources

## Security Questions

### Is Relyr secure?

Yes, Relyr includes several security features:
- TLS/SSL encryption
- Authentication mechanisms
- Access control lists
- Request/response filtering
- Rate limiting

### How do I secure my Relyr installation?

1. **Enable authentication:**
   ```yaml
   auth:
     enabled: true
   ```

2. **Use HTTPS:**
   ```yaml
   server:
     tls:
       enabled: true
   ```

3. **Configure firewall:**
   ```bash
   sudo ufw allow from 192.168.1.0/24 to any port 8080
   ```

4. **Regular updates:**
   ```bash
   relyr version --check-updates
   ```

### Can I restrict access by IP address?

Yes, configure IP whitelisting:

```yaml
security:
  ip_whitelist:
    - 192.168.1.0/24
    - 10.0.0.0/8
  deny_by_default: true
```

### How are passwords stored?

Passwords are hashed using bcrypt with a configurable cost factor:

```yaml
auth:
  password_hashing:
    algorithm: bcrypt
    cost: 12
```

## API Questions

### Does Relyr have an API?

Yes, Relyr provides a comprehensive REST API for management and monitoring. See our [API Documentation](../api/overview.md).

### How do I authenticate with the API?

Use API keys, bearer tokens, or basic authentication:

```bash
# API Key
curl -H "X-API-Key: your-api-key" http://localhost:8080/api/v1/proxies

# Bearer Token
curl -H "Authorization: Bearer your-token" http://localhost:8080/api/v1/proxies
```

### Are there SDKs available?

Yes, official SDKs are available for:
- Python: `pip install relyr-python`
- Node.js: `npm install relyr-node`
- Go: `go get github.com/relyr/relyr-go`

## Monitoring Questions

### How do I monitor Relyr?

Relyr provides multiple monitoring options:

1. **Built-in health checks:**
   ```bash
   curl http://localhost:8080/health
   ```

2. **Metrics endpoint:**
   ```bash
   curl http://localhost:8080/metrics
   ```

3. **API endpoints:**
   ```bash
   curl -H "X-API-Key: key" http://localhost:8080/api/v1/status
   ```

### What metrics are available?

Key metrics include:
- Requests per second
- Response times (avg, p95, p99)
- Active connections
- Error rates
- Bandwidth usage
- Memory/CPU usage

### Can I integrate with monitoring systems?

Yes, Relyr supports:
- Prometheus metrics
- StatsD integration
- Custom webhooks
- Syslog output

```yaml
monitoring:
  prometheus:
    enabled: true
    port: 9090
  statsd:
    enabled: true
    host: statsd.example.com
    port: 8125
```

## Docker Questions

### How do I run Relyr in Docker?

```bash
# Basic usage
docker run -d -p 8080:8080 relyr/relyr:latest

# With custom configuration
docker run -d -p 8080:8080 \
  -v $(pwd)/config:/etc/relyr \
  relyr/relyr:latest
```

### Can I use Docker Compose?

Yes, here's a sample `docker-compose.yml`:

```yaml
version: '3.8'
services:
  relyr:
    image: relyr/relyr:latest
    ports:
      - "8080:8080"
    volumes:
      - ./config:/etc/relyr
    environment:
      - RELYR_LOG_LEVEL=info
```

### How do I persist data in Docker?

Mount volumes for configuration and data:

```bash
docker run -d \
  -v relyr-config:/etc/relyr \
  -v relyr-data:/var/lib/relyr \
  -v relyr-logs:/var/log/relyr \
  relyr/relyr:latest
```

## Kubernetes Questions

### Can I deploy Relyr on Kubernetes?

Yes, Relyr works well in Kubernetes. See our [Kubernetes examples](../examples/advanced.md#kubernetes-deployment).

### How do I scale Relyr in Kubernetes?

Use horizontal pod autoscaling:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: relyr-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: relyr
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

## Troubleshooting Questions

### Relyr won't start. What should I check?

1. **Check logs:**
   ```bash
   journalctl -u relyr -f
   ```

2. **Validate configuration:**
   ```bash
   relyr config validate
   ```

3. **Check port availability:**
   ```bash
   netstat -tulpn | grep :8080
   ```

4. **Verify permissions:**
   ```bash
   ls -la ~/.config/relyr/
   ```

### Why are my proxy connections slow?

Common causes and solutions:

1. **Network latency:** Test direct connection
2. **DNS issues:** Configure DNS servers
3. **Upstream proxy problems:** Test upstream directly
4. **Resource constraints:** Monitor CPU/memory usage

### How do I enable debug logging?

Update your configuration:

```yaml
logging:
  level: debug
  output: stdout
```

Or use environment variable:
```bash
export RELYR_LOG_LEVEL=debug
relyr server
```

## Integration Questions

### Can I use Relyr with existing applications?

Yes, Relyr works with any application that supports HTTP/HTTPS proxies. Configure your application to use Relyr as a proxy server.

### Does Relyr work with load balancers?

Yes, you can place Relyr behind load balancers like:
- nginx
- HAProxy
- AWS Application Load Balancer
- Google Cloud Load Balancer

### Can I integrate with CI/CD pipelines?

Yes, Relyr can be used in CI/CD for:
- Testing through proxies
- Accessing internal resources
- Network security testing

Example GitHub Actions:
```yaml
- name: Start Relyr
  run: |
    docker run -d -p 8080:8080 relyr/relyr:latest
    sleep 5

- name: Test through proxy
  run: |
    curl -x http://localhost:8080 https://api.example.com
```

## Licensing Questions

### What license does Relyr use?

Relyr is released under the MIT License, which allows:
- Commercial use
- Distribution
- Modification
- Private use

### Can I use Relyr in commercial projects?

Yes, the MIT license allows commercial use without restrictions.

### Do I need to pay for Relyr?

The open-source version is completely free. Enterprise support and additional features may be available through commercial licenses.

## Getting Help

### Where can I get support?

1. **Documentation:** This site and [GitHub Wiki](https://github.com/relyr/relyr/wiki)
2. **Issues:** [GitHub Issues](https://github.com/relyr/relyr/issues) for bugs and feature requests
3. **Community:** [Community Forum](https://community.relyr.io) for discussions
4. **Chat:** [Discord Server](https://discord.gg/relyr) for real-time help

### How do I report a bug?

1. Check existing [GitHub Issues](https://github.com/relyr/relyr/issues)
2. Create a new issue with:
   - Relyr version (`relyr version`)
   - Operating system
   - Configuration file (sanitized)
   - Error logs
   - Steps to reproduce

### How do I request a feature?

1. Check if it's already requested in [GitHub Issues](https://github.com/relyr/relyr/issues)
2. Create a feature request with:
   - Clear description of the feature
   - Use case/motivation
   - Proposed implementation (if applicable)

### Is commercial support available?

Yes, commercial support options include:
- Priority support
- Custom feature development
- Training and consulting
- SLA guarantees

Contact us at [support@relyr.io](mailto:support@relyr.io) for more information.
