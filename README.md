# flame Installation Guide

flame is a free and open-source self-hosted startpage for your server. Flame provides a beautiful, customizable dashboard for self-hosted services, serving as an alternative to Heimdall or Homer

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
  - CPU: 1 core minimum
  - RAM: 128MB minimum
  - Storage: 100MB for application
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 5005 (default flame port)
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

# Install flame
sudo dnf install -y flame

# Enable and start service
sudo systemctl enable --now flame

# Configure firewall
sudo firewall-cmd --permanent --add-port=5005/tcp
sudo firewall-cmd --reload

# Verify installation
flame --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install flame
sudo apt install -y flame

# Enable and start service
sudo systemctl enable --now flame

# Configure firewall
sudo ufw allow 5005

# Verify installation
flame --version
```

### Arch Linux

```bash
# Install flame
sudo pacman -S flame

# Enable and start service
sudo systemctl enable --now flame

# Verify installation
flame --version
```

### Alpine Linux

```bash
# Install flame
apk add --no-cache flame

# Enable and start service
rc-update add flame default
rc-service flame start

# Verify installation
flame --version
```

### openSUSE/SLES

```bash
# Install flame
sudo zypper install -y flame

# Enable and start service
sudo systemctl enable --now flame

# Configure firewall
sudo firewall-cmd --permanent --add-port=5005/tcp
sudo firewall-cmd --reload

# Verify installation
flame --version
```

### macOS

```bash
# Using Homebrew
brew install flame

# Start service
brew services start flame

# Verify installation
flame --version
```

### FreeBSD

```bash
# Using pkg
pkg install flame

# Enable in rc.conf
echo 'flame_enable="YES"' >> /etc/rc.conf

# Start service
service flame start

# Verify installation
flame --version
```

### Windows

```bash
# Using Chocolatey
choco install flame

# Or using Scoop
scoop install flame

# Verify installation
flame --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/flame

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
flame --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable flame

# Start service
sudo systemctl start flame

# Stop service
sudo systemctl stop flame

# Restart service
sudo systemctl restart flame

# Check status
sudo systemctl status flame

# View logs
sudo journalctl -u flame -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add flame default

# Start service
rc-service flame start

# Stop service
rc-service flame stop

# Restart service
rc-service flame restart

# Check status
rc-service flame status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'flame_enable="YES"' >> /etc/rc.conf

# Start service
service flame start

# Stop service
service flame stop

# Restart service
service flame restart

# Check status
service flame status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start flame
brew services stop flame
brew services restart flame

# Check status
brew services list | grep flame
```

### Windows Service Manager

```powershell
# Start service
net start flame

# Stop service
net stop flame

# Using PowerShell
Start-Service flame
Stop-Service flame
Restart-Service flame

# Check status
Get-Service flame
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream flame_backend {
    server 127.0.0.1:5005;
}

server {
    listen 80;
    server_name flame.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name flame.example.com;

    ssl_certificate /etc/ssl/certs/flame.example.com.crt;
    ssl_certificate_key /etc/ssl/private/flame.example.com.key;

    location / {
        proxy_pass http://flame_backend;
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
    ServerName flame.example.com
    Redirect permanent / https://flame.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName flame.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/flame.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/flame.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:5005/
    ProxyPassReverse / http://127.0.0.1:5005/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend flame_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/flame.pem
    redirect scheme https if !{ ssl_fc }
    default_backend flame_backend

backend flame_backend
    balance roundrobin
    server flame1 127.0.0.1:5005 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R flame:flame /etc/flame
sudo chmod 750 /etc/flame

# Configure firewall
sudo firewall-cmd --permanent --add-port=5005/tcp
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
sudo systemctl status flame

# View logs
sudo journalctl -u flame -f

# Monitor resource usage
top -p $(pgrep flame)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/flame"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/flame-backup-$DATE.tar.gz" /etc/flame /var/lib/flame

echo "Backup completed: $BACKUP_DIR/flame-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop flame

# Restore from backup
tar -xzf /backup/flame/flame-backup-*.tar.gz -C /

# Start service
sudo systemctl start flame
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u flame -n 100
sudo tail -f /var/log/flame/flame.log

# Check configuration
flame --version

# Check permissions
ls -la /etc/flame
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 5005

# Test connectivity
telnet localhost 5005

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep flame)

# Check disk I/O
iotop -p $(pgrep flame)

# Check connections
ss -an | grep 5005
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  flame:
    image: flame:latest
    ports:
      - "5005:5005"
    volumes:
      - ./config:/etc/flame
      - ./data:/var/lib/flame
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update flame

# Debian/Ubuntu
sudo apt update && sudo apt upgrade flame

# Arch Linux
sudo pacman -Syu flame

# Alpine Linux
apk update && apk upgrade flame

# openSUSE
sudo zypper update flame

# FreeBSD
pkg update && pkg upgrade flame

# Always backup before updates
tar -czf /backup/flame-pre-update-$(date +%Y%m%d).tar.gz /etc/flame

# Restart after updates
sudo systemctl restart flame
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/flame

# Clean old logs
find /var/log/flame -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/flame
```

## Additional Resources

- Official Documentation: https://docs.flame.org/
- GitHub Repository: https://github.com/flame/flame
- Community Forum: https://forum.flame.org/
- Best Practices Guide: https://docs.flame.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
