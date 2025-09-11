# Installation Guide

This guide covers various methods to install and set up Relyr on different platforms.

## System Requirements

### Minimum Requirements

- **Operating System**: Linux, macOS, or Windows
- **Memory**: 512MB RAM (2GB recommended)
- **Storage**: 100MB free space
- **Network**: Internet connection for initial setup

### Recommended Requirements

- **Memory**: 4GB RAM or more
- **CPU**: 2+ cores
- **Storage**: 1GB free space
- **Network**: Stable broadband connection

## Installation Methods

### Package Managers

=== "pip (Python)"

    ```bash
    # Install from PyPI
    pip install relyr
    
    # Install with specific extras
    pip install relyr[proxy,monitoring,dev]
    
    # Install from source
    pip install git+https://github.com/relyr/relyr.git
    ```

=== "npm (Node.js)"

    ```bash
    # Install globally
    npm install -g relyr
    
    # Install locally
    npm install relyr
    
    # Install with yarn
    yarn add relyr
    ```

=== "Homebrew (macOS)"

    ```bash
    # Add Relyr tap
    brew tap relyr/tap
    
    # Install Relyr
    brew install relyr
    
    # Upgrade to latest version
    brew upgrade relyr
    ```

=== "APT (Ubuntu/Debian)"

    ```bash
    # Add Relyr repository
    curl -fsSL https://packages.relyr.io/gpg | sudo apt-key add -
    echo "deb https://packages.relyr.io/apt stable main" | sudo tee /etc/apt/sources.list.d/relyr.list
    
    # Update package list
    sudo apt update
    
    # Install Relyr
    sudo apt install relyr
    ```

=== "YUM (CentOS/RHEL)"

    ```bash
    # Add Relyr repository
    sudo tee /etc/yum.repos.d/relyr.repo <<EOF
    [relyr]
    name=Relyr Repository
    baseurl=https://packages.relyr.io/yum/
    enabled=1
    gpgcheck=1
    gpgkey=https://packages.relyr.io/gpg
    EOF
    
    # Install Relyr
    sudo yum install relyr
    ```

### Binary Downloads

Download pre-built binaries from the [releases page](https://github.com/relyr/relyr/releases):

=== "Linux"

    ```bash
    # Download and install
    curl -LO https://github.com/relyr/relyr/releases/latest/download/relyr-linux-amd64.tar.gz
    tar -xzf relyr-linux-amd64.tar.gz
    sudo mv relyr /usr/local/bin/
    
    # Verify installation
    relyr --version
    ```

=== "macOS"

    ```bash
    # Download and install
    curl -LO https://github.com/relyr/relyr/releases/latest/download/relyr-darwin-amd64.tar.gz
    tar -xzf relyr-darwin-amd64.tar.gz
    sudo mv relyr /usr/local/bin/
    
    # Verify installation
    relyr --version
    ```

=== "Windows"

    ```powershell
    # Download using PowerShell
    Invoke-WebRequest -Uri "https://github.com/relyr/relyr/releases/latest/download/relyr-windows-amd64.zip" -OutFile "relyr.zip"
    
    # Extract and install
    Expand-Archive -Path "relyr.zip" -DestinationPath "C:\Program Files\Relyr"
    
    # Add to PATH (run as Administrator)
    [Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Relyr", [EnvironmentVariableTarget]::Machine)
    ```

### Container Installation

=== "Docker"

    ```bash
    # Pull the official image
    docker pull relyr/relyr:latest
    
    # Run Relyr container
    docker run -d \
      --name relyr \
      -p 8080:8080 \
      -v $(pwd)/config:/etc/relyr \
      relyr/relyr:latest
    
    # Run with custom configuration
    docker run -d \
      --name relyr \
      -p 8080:8080 \
      -e RELYR_LOG_LEVEL=debug \
      -v $(pwd)/relyr.yml:/etc/relyr/relyr.yml \
      relyr/relyr:latest
    ```

=== "Docker Compose"

    Create a `docker-compose.yml` file:

    ```yaml
    version: '3.8'
    
    services:
      relyr:
        image: relyr/relyr:latest
        container_name: relyr
        ports:
          - "8080:8080"
          - "8443:8443"
        volumes:
          - ./config:/etc/relyr
          - ./data:/var/lib/relyr
        environment:
          - RELYR_LOG_LEVEL=info
          - RELYR_CONFIG_PATH=/etc/relyr/relyr.yml
        restart: unless-stopped
        networks:
          - relyr-network
    
    networks:
      relyr-network:
        driver: bridge
    ```

    Run with:

    ```bash
    docker-compose up -d
    ```

=== "Kubernetes"

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: relyr
      namespace: default
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: relyr
      template:
        metadata:
          labels:
            app: relyr
        spec:
          containers:
          - name: relyr
            image: relyr/relyr:latest
            ports:
            - containerPort: 8080
            env:
            - name: RELYR_LOG_LEVEL
              value: "info"
            volumeMounts:
            - name: config
              mountPath: /etc/relyr
          volumes:
          - name: config
            configMap:
              name: relyr-config
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: relyr-service
    spec:
      selector:
        app: relyr
      ports:
      - protocol: TCP
        port: 80
        targetPort: 8080
      type: LoadBalancer
    ```

## Post-Installation Setup

### Initial Configuration

1. **Create configuration directory**:

    ```bash
    mkdir -p ~/.config/relyr
    ```

2. **Generate default configuration**:

    ```bash
    relyr config init
    ```

3. **Edit configuration file**:

    ```bash
    # Open in your preferred editor
    nano ~/.config/relyr/relyr.yml
    ```

### Service Setup

=== "systemd (Linux)"

    Create a systemd service file:

    ```bash
    sudo tee /etc/systemd/system/relyr.service <<EOF
    [Unit]
    Description=Relyr Proxy Service
    After=network.target
    
    [Service]
    Type=simple
    User=relyr
    Group=relyr
    ExecStart=/usr/local/bin/relyr server
    Restart=on-failure
    RestartSec=5
    Environment=RELYR_CONFIG_PATH=/etc/relyr/relyr.yml
    
    [Install]
    WantedBy=multi-user.target
    EOF
    
    # Enable and start service
    sudo systemctl enable relyr
    sudo systemctl start relyr
    ```

=== "launchd (macOS)"

    Create a launch daemon:

    ```bash
    sudo tee /Library/LaunchDaemons/io.relyr.plist <<EOF
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
        <key>Label</key>
        <string>io.relyr</string>
        <key>ProgramArguments</key>
        <array>
            <string>/usr/local/bin/relyr</string>
            <string>server</string>
        </array>
        <key>RunAtLoad</key>
        <true/>
        <key>KeepAlive</key>
        <true/>
    </dict>
    </plist>
    EOF
    
    # Load and start service
    sudo launchctl load /Library/LaunchDaemons/io.relyr.plist
    ```

=== "Windows Service"

    Install as Windows service:

    ```powershell
    # Install service (run as Administrator)
    relyr service install
    
    # Start service
    relyr service start
    
    # Check service status
    relyr service status
    ```

### User and Permissions

Create a dedicated user for Relyr:

```bash
# Create user and group
sudo useradd -r -s /bin/false relyr
sudo mkdir -p /etc/relyr /var/lib/relyr /var/log/relyr

# Set permissions
sudo chown -R relyr:relyr /etc/relyr /var/lib/relyr /var/log/relyr
sudo chmod 755 /etc/relyr /var/lib/relyr /var/log/relyr
```

## Verification

### Check Installation

```bash
# Verify binary installation
relyr --version
relyr --help

# Check configuration
relyr config validate

# Test connectivity
relyr health-check
```

### Basic Functionality Test

```bash
# Start Relyr in foreground mode
relyr server --config /etc/relyr/relyr.yml

# In another terminal, test proxy
curl -x http://localhost:8080 http://httpbin.org/ip
```

## Development Installation

For development and testing:

### From Source

```bash
# Clone repository
git clone https://github.com/relyr/relyr.git
cd relyr

# Install dependencies
make deps

# Build from source
make build

# Run tests
make test

# Install locally
make install
```

### Development Environment

```bash
# Create virtual environment (Python)
python -m venv venv
source venv/bin/activate  # Linux/macOS
# or
venv\Scripts\activate  # Windows

# Install in development mode
pip install -e .

# Install development dependencies
pip install -r requirements-dev.txt
```

## Troubleshooting Installation

### Common Issues

1. **Permission denied errors**:
   ```bash
   sudo chown -R $USER:$USER ~/.config/relyr
   ```

2. **Port already in use**:
   ```bash
   # Check what's using the port
   sudo netstat -tulpn | grep :8080
   
   # Kill process if needed
   sudo kill -9 <PID>
   ```

3. **Missing dependencies**:
   ```bash
   # Ubuntu/Debian
   sudo apt install build-essential curl wget
   
   # CentOS/RHEL
   sudo yum groupinstall "Development Tools"
   sudo yum install curl wget
   ```

### Getting Help

- Check the [Troubleshooting Guide](../troubleshooting/common-issues.md)
- Visit our [GitHub Issues](https://github.com/relyr/relyr/issues)
- Join our [Community Forum](https://community.relyr.io)

## Next Steps

After installation, check out:

- [Quick Start Guide](quickstart.md) - Get up and running quickly
- [Proxy Setup](proxy-setup.md) - Configure HTTP proxies
- [API Reference](api/overview.md) - Explore the API
- [Examples](examples/basic.md) - See practical examples
