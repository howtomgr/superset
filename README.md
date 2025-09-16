# superset Installation Guide

superset is a free and open-source data exploration. Superset provides modern data exploration and visualization platform

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 2+ cores
  - RAM: 4GB minimum
  - Storage: 5GB for data
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8088 (default superset port)
  - None
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install superset
sudo dnf install -y superset

# Enable and start service
sudo systemctl enable --now superset

# Configure firewall
sudo firewall-cmd --permanent --add-port=8088/tcp
sudo firewall-cmd --reload

# Verify installation
superset --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install superset
sudo apt install -y superset

# Enable and start service
sudo systemctl enable --now superset

# Configure firewall
sudo ufw allow 8088

# Verify installation
superset --version
```

### Arch Linux

```bash
# Install superset
sudo pacman -S superset

# Enable and start service
sudo systemctl enable --now superset

# Verify installation
superset --version
```

### Alpine Linux

```bash
# Install superset
apk add --no-cache superset

# Enable and start service
rc-update add superset default
rc-service superset start

# Verify installation
superset --version
```

### openSUSE/SLES

```bash
# Install superset
sudo zypper install -y superset

# Enable and start service
sudo systemctl enable --now superset

# Configure firewall
sudo firewall-cmd --permanent --add-port=8088/tcp
sudo firewall-cmd --reload

# Verify installation
superset --version
```

### macOS

```bash
# Using Homebrew
brew install superset

# Start service
brew services start superset

# Verify installation
superset --version
```

### FreeBSD

```bash
# Using pkg
pkg install superset

# Enable in rc.conf
echo 'superset_enable="YES"' >> /etc/rc.conf

# Start service
service superset start

# Verify installation
superset --version
```

### Windows

```bash
# Using Chocolatey
choco install superset

# Or using Scoop
scoop install superset

# Verify installation
superset --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/superset

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
superset --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable superset

# Start service
sudo systemctl start superset

# Stop service
sudo systemctl stop superset

# Restart service
sudo systemctl restart superset

# Check status
sudo systemctl status superset

# View logs
sudo journalctl -u superset -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add superset default

# Start service
rc-service superset start

# Stop service
rc-service superset stop

# Restart service
rc-service superset restart

# Check status
rc-service superset status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'superset_enable="YES"' >> /etc/rc.conf

# Start service
service superset start

# Stop service
service superset stop

# Restart service
service superset restart

# Check status
service superset status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start superset
brew services stop superset
brew services restart superset

# Check status
brew services list | grep superset
```

### Windows Service Manager

```powershell
# Start service
net start superset

# Stop service
net stop superset

# Using PowerShell
Start-Service superset
Stop-Service superset
Restart-Service superset

# Check status
Get-Service superset
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream superset_backend {
    server 127.0.0.1:8088;
}

server {
    listen 80;
    server_name superset.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name superset.example.com;

    ssl_certificate /etc/ssl/certs/superset.example.com.crt;
    ssl_certificate_key /etc/ssl/private/superset.example.com.key;

    location / {
        proxy_pass http://superset_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName superset.example.com
    Redirect permanent / https://superset.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName superset.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/superset.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/superset.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8088/
    ProxyPassReverse / http://127.0.0.1:8088/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend superset_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/superset.pem
    redirect scheme https if !{ ssl_fc }
    default_backend superset_backend

backend superset_backend
    balance roundrobin
    server superset1 127.0.0.1:8088 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R superset:superset /etc/superset
sudo chmod 750 /etc/superset

# Configure firewall
sudo firewall-cmd --permanent --add-port=8088/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status superset

# View logs
sudo journalctl -u superset -f

# Monitor resource usage
top -p $(pgrep superset)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/superset"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/superset-backup-$DATE.tar.gz" /etc/superset /var/lib/superset

echo "Backup completed: $BACKUP_DIR/superset-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop superset

# Restore from backup
tar -xzf /backup/superset/superset-backup-*.tar.gz -C /

# Start service
sudo systemctl start superset
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u superset -n 100
sudo tail -f /var/log/superset/superset.log

# Check configuration
superset --version

# Check permissions
ls -la /etc/superset
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8088

# Test connectivity
telnet localhost 8088

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep superset)

# Check disk I/O
iotop -p $(pgrep superset)

# Check connections
ss -an | grep 8088
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  superset:
    image: superset:latest
    ports:
      - "8088:8088"
    volumes:
      - ./config:/etc/superset
      - ./data:/var/lib/superset
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update superset

# Debian/Ubuntu
sudo apt update && sudo apt upgrade superset

# Arch Linux
sudo pacman -Syu superset

# Alpine Linux
apk update && apk upgrade superset

# openSUSE
sudo zypper update superset

# FreeBSD
pkg update && pkg upgrade superset

# Always backup before updates
tar -czf /backup/superset-pre-update-$(date +%Y%m%d).tar.gz /etc/superset

# Restart after updates
sudo systemctl restart superset
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/superset

# Clean old logs
find /var/log/superset -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/superset
```

## Additional Resources

- Official Documentation: https://docs.superset.org/
- GitHub Repository: https://github.com/superset/superset
- Community Forum: https://forum.superset.org/
- Best Practices Guide: https://docs.superset.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
