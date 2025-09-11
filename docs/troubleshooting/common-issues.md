# Common Issues & Solutions

This guide covers the most frequently encountered issues when using Relyr and their solutions.

## Installation Issues

### Permission Denied Errors

**Problem:** Getting permission denied when installing or running Relyr.

**Solutions:**

1. **Linux/macOS - Binary Installation:**
   ```bash
   # Fix permissions for binary
   sudo chmod +x /usr/local/bin/relyr
   
   # Fix ownership
   sudo chown root:root /usr/local/bin/relyr
   ```

2. **Configuration Directory:**
   ```bash
   # Create and fix config directory permissions
   mkdir -p ~/.config/relyr
   chmod 755 ~/.config/relyr
   ```

3. **Service Installation:**
   ```bash
   # For systemd services
   sudo systemctl daemon-reload
   sudo systemctl enable relyr
   ```

### Missing Dependencies

**Problem:** Relyr fails to start due to missing system dependencies.

**Solutions:**

1. **Ubuntu/Debian:**
   ```bash
   sudo apt update
   sudo apt install build-essential curl wget ca-certificates
   ```

2. **CentOS/RHEL:**
   ```bash
   sudo yum update
   sudo yum groupinstall "Development Tools"
   sudo yum install curl wget ca-certificates
   ```

3. **macOS:**
   ```bash
   # Install Xcode command line tools
   xcode-select --install
   
   # Or using Homebrew
   brew install curl wget
   ```

### Package Manager Issues

**Problem:** Package manager can't find Relyr packages.

**Solutions:**

1. **Update package sources:**
   ```bash
   # APT
   sudo apt update
   
   # YUM
   sudo yum clean all && sudo yum update
   
   # Homebrew
   brew update
   ```

2. **Verify repository configuration:**
   ```bash
   # Check if Relyr repository is added
   cat /etc/apt/sources.list.d/relyr.list
   ```

## Configuration Issues

### Invalid Configuration File

**Problem:** Relyr fails to start with configuration errors.

**Diagnosis:**
```bash
# Validate configuration
relyr config validate

# Check configuration syntax
relyr config check --file /path/to/relyr.yml
```

**Solutions:**

1. **YAML Syntax Errors:**
   ```bash
   # Use a YAML validator
   python -c "import yaml; yaml.safe_load(open('/path/to/relyr.yml'))"
   ```

2. **Common YAML mistakes:**
   ```yaml
   # ❌ Wrong indentation
   server:
   host: 0.0.0.0
   port: 8080
   
   # ✅ Correct indentation
   server:
     host: 0.0.0.0
     port: 8080
   
   # ❌ Missing quotes for special characters
   password: p@ssw0rd!
   
   # ✅ Quoted special characters
   password: "p@ssw0rd!"
   ```

### Port Already in Use

**Problem:** Relyr can't bind to the configured port.

**Diagnosis:**
```bash
# Check what's using the port
sudo netstat -tulpn | grep :8080
sudo lsof -i :8080

# On macOS
lsof -i :8080
```

**Solutions:**

1. **Change port in configuration:**
   ```yaml
   server:
     port: 8081  # Use a different port
   ```

2. **Kill conflicting process:**
   ```bash
   # Find and kill the process
   sudo kill -9 <PID>
   
   # Or stop the service
   sudo systemctl stop <service-name>
   ```

3. **Use a different port:**
   ```bash
   # Start with custom port
   relyr server --port 8081
   ```

## Connection Issues

### Proxy Connection Failures

**Problem:** Clients can't connect through the proxy.

**Diagnosis:**
```bash
# Test proxy connectivity
curl -x http://localhost:8080 https://httpbin.org/ip

# Check if Relyr is listening
netstat -tulpn | grep :8080

# Test with verbose output
curl -v -x http://localhost:8080 https://httpbin.org/ip
```

**Solutions:**

1. **Firewall blocking connections:**
   ```bash
   # Ubuntu/Debian
   sudo ufw allow 8080
   
   # CentOS/RHEL
   sudo firewall-cmd --add-port=8080/tcp --permanent
   sudo firewall-cmd --reload
   
   # Check iptables
   sudo iptables -L -n
   ```

2. **Binding to wrong interface:**
   ```yaml
   server:
     host: 0.0.0.0  # Listen on all interfaces
     port: 8080
   ```

3. **Authentication issues:**
   ```bash
   # Test with authentication
   curl -x http://username:password@localhost:8080 https://httpbin.org/ip
   ```

### SSL/TLS Certificate Issues

**Problem:** HTTPS proxy connections fail with certificate errors.

**Diagnosis:**
```bash
# Test SSL connection
openssl s_client -connect localhost:8443 -servername localhost

# Check certificate validity
openssl x509 -in /path/to/cert.pem -text -noout
```

**Solutions:**

1. **Self-signed certificate issues:**
   ```bash
   # Generate new self-signed certificate
   openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
   ```

2. **Certificate path configuration:**
   ```yaml
   server:
     tls:
       enabled: true
       cert_file: /path/to/cert.pem
       key_file: /path/to/key.pem
   ```

3. **Trust certificate on client:**
   ```bash
   # Add to system trust store (Ubuntu)
   sudo cp cert.pem /usr/local/share/ca-certificates/relyr.crt
   sudo update-ca-certificates
   ```

### Upstream Proxy Issues

**Problem:** Connection to upstream proxy fails.

**Diagnosis:**
```bash
# Test upstream proxy directly
curl -x http://upstream.proxy.com:8080 https://httpbin.org/ip

# Check DNS resolution
nslookup upstream.proxy.com
dig upstream.proxy.com
```

**Solutions:**

1. **DNS resolution issues:**
   ```yaml
   upstream:
     proxy: http://192.168.1.100:8080  # Use IP instead of hostname
   ```

2. **Authentication with upstream:**
   ```yaml
   upstream:
     proxy: http://upstream.proxy.com:8080
     auth:
       username: "user"
       password: "pass"
   ```

3. **Timeout configuration:**
   ```yaml
   upstream:
     proxy: http://upstream.proxy.com:8080
     timeout: 60s
   ```

## Performance Issues

### High Memory Usage

**Problem:** Relyr consuming too much memory.

**Diagnosis:**
```bash
# Check memory usage
ps aux | grep relyr
top -p $(pgrep relyr)

# Check for memory leaks
valgrind --tool=memcheck --leak-check=full relyr server
```

**Solutions:**

1. **Limit connection pool size:**
   ```yaml
   performance:
     max_connections: 500
     max_connections_per_host: 50
   ```

2. **Adjust garbage collection (if applicable):**
   ```bash
   # For Go-based builds
   export GOGC=100
   ```

3. **Monitor and restart if needed:**
   ```bash
   # Add to systemd service
   MemoryMax=1G
   Restart=on-failure
   ```

### Slow Response Times

**Problem:** Proxy responses are slow.

**Diagnosis:**
```bash
# Measure response times
time curl -x http://localhost:8080 https://httpbin.org/ip

# Check network latency
ping target-server.com
traceroute target-server.com
```

**Solutions:**

1. **Optimize timeouts:**
   ```yaml
   proxy:
     timeouts:
       connect: 5s
       read: 30s
       write: 30s
   ```

2. **Enable connection pooling:**
   ```yaml
   performance:
     connection_pooling:
       enabled: true
       max_idle_connections: 100
       idle_timeout: 90s
   ```

3. **Use compression:**
   ```yaml
   proxy:
     compression:
       enabled: true
       level: 6
   ```

### High CPU Usage

**Problem:** Relyr using excessive CPU resources.

**Solutions:**

1. **Limit concurrent connections:**
   ```yaml
   performance:
     max_concurrent_requests: 1000
     worker_threads: 4
   ```

2. **Optimize logging:**
   ```yaml
   logging:
     level: warn  # Reduce log verbosity
     async: true
   ```

3. **Profile CPU usage:**
   ```bash
   # Use profiling tools
   perf top -p $(pgrep relyr)
   ```

## Authentication Issues

### API Key Not Working

**Problem:** API requests fail with authentication errors.

**Solutions:**

1. **Verify API key format:**
   ```bash
   # Check key format
   echo $RELYR_API_KEY | cut -c1-10
   ```

2. **Check key permissions:**
   ```bash
   relyr api-key list
   relyr api-key show --id key_123456
   ```

3. **Test with curl:**
   ```bash
   curl -H "X-API-Key: $RELYR_API_KEY" \
        http://localhost:8080/api/v1/health
   ```

### Token Expiration

**Problem:** Bearer tokens expire too quickly.

**Solutions:**

1. **Increase token lifetime:**
   ```yaml
   auth:
     jwt:
       access_token_ttl: 3600  # 1 hour
       refresh_token_ttl: 86400  # 24 hours
   ```

2. **Implement token refresh:**
   ```python
   # Python example
   def refresh_token():
       response = requests.post('/api/v1/auth/refresh', 
                               json={'refresh_token': refresh_token})
       return response.json()['access_token']
   ```

## Logging Issues

### Missing Log Files

**Problem:** Log files are not being created.

**Solutions:**

1. **Check log directory permissions:**
   ```bash
   sudo mkdir -p /var/log/relyr
   sudo chown relyr:relyr /var/log/relyr
   sudo chmod 755 /var/log/relyr
   ```

2. **Verify logging configuration:**
   ```yaml
   logging:
     level: info
     output: file
     file: /var/log/relyr/relyr.log
     rotation:
       enabled: true
       max_size: 100MB
       max_files: 10
   ```

### Log Rotation Not Working

**Problem:** Log files grow too large.

**Solutions:**

1. **Configure logrotate:**
   ```bash
   sudo tee /etc/logrotate.d/relyr <<EOF
   /var/log/relyr/*.log {
       daily
       missingok
       rotate 30
       compress
       notifempty
       create 644 relyr relyr
       postrotate
           systemctl reload relyr
       endscript
   }
   EOF
   ```

## Network Issues

### DNS Resolution Problems

**Problem:** Proxy can't resolve hostnames.

**Solutions:**

1. **Configure DNS servers:**
   ```yaml
   network:
     dns:
       servers:
         - 8.8.8.8
         - 1.1.1.1
       timeout: 5s
   ```

2. **Test DNS resolution:**
   ```bash
   nslookup google.com 8.8.8.8
   dig @1.1.1.1 google.com
   ```

### IPv6 Issues

**Problem:** IPv6 connections fail.

**Solutions:**

1. **Disable IPv6 if not needed:**
   ```yaml
   network:
     ipv6:
       enabled: false
   ```

2. **Configure IPv6 properly:**
   ```yaml
   network:
     ipv6:
       enabled: true
       prefer_ipv4: true
   ```

## Docker Issues

### Container Won't Start

**Problem:** Relyr Docker container fails to start.

**Solutions:**

1. **Check container logs:**
   ```bash
   docker logs relyr
   docker logs --follow relyr
   ```

2. **Verify port mapping:**
   ```bash
   docker run -p 8080:8080 relyr/relyr:latest
   ```

3. **Check volume mounts:**
   ```bash
   docker run -v $(pwd)/config:/etc/relyr relyr/relyr:latest
   ```

### Permission Issues in Container

**Problem:** Permission denied errors in Docker container.

**Solutions:**

1. **Run as non-root user:**
   ```dockerfile
   USER 1000:1000
   ```

2. **Fix volume permissions:**
   ```bash
   sudo chown -R 1000:1000 ./config
   ```

## Getting Help

If you can't resolve your issue:

1. **Check system logs:**
   ```bash
   # systemd logs
   journalctl -u relyr -f
   
   # Application logs
   tail -f /var/log/relyr/relyr.log
   ```

2. **Enable debug logging:**
   ```yaml
   logging:
     level: debug
   ```

3. **Collect system information:**
   ```bash
   relyr version
   relyr config show
   uname -a
   ```

4. **Report the issue:**
   - [GitHub Issues](https://github.com/relyr/relyr/issues)
   - [Community Forum](https://community.relyr.io)
   - Include logs, configuration, and system info

## Preventive Measures

### Regular Maintenance

1. **Update regularly:**
   ```bash
   # Check for updates
   relyr version --check-updates
   
   # Update using package manager
   sudo apt update && sudo apt upgrade relyr
   ```

2. **Monitor resource usage:**
   ```bash
   # Set up monitoring
   watch 'ps aux | grep relyr'
   ```

3. **Backup configuration:**
   ```bash
   cp ~/.config/relyr/relyr.yml ~/.config/relyr/relyr.yml.backup
   ```

### Health Checks

Set up automated health checks:

```bash
#!/bin/bash
# health-check.sh
if ! curl -f http://localhost:8080/health > /dev/null 2>&1; then
    echo "Relyr health check failed"
    systemctl restart relyr
fi
```

Add to crontab:
```bash
# Run every 5 minutes
*/5 * * * * /path/to/health-check.sh
```
