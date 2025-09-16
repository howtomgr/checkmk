# checkmk Installation Guide

checkmk is a free and open-source IT monitoring. Checkmk provides comprehensive IT monitoring solution

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
  - Storage: 10GB for data
  - Network: HTTP/agent access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default checkmk port)
  - Agent on 6556
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

# Install checkmk
sudo dnf install -y checkmk

# Enable and start service
sudo systemctl enable --now checkmk

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
checkmk --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install checkmk
sudo apt install -y checkmk

# Enable and start service
sudo systemctl enable --now checkmk

# Configure firewall
sudo ufw allow 80

# Verify installation
checkmk --version
```

### Arch Linux

```bash
# Install checkmk
sudo pacman -S checkmk

# Enable and start service
sudo systemctl enable --now checkmk

# Verify installation
checkmk --version
```

### Alpine Linux

```bash
# Install checkmk
apk add --no-cache checkmk

# Enable and start service
rc-update add checkmk default
rc-service checkmk start

# Verify installation
checkmk --version
```

### openSUSE/SLES

```bash
# Install checkmk
sudo zypper install -y checkmk

# Enable and start service
sudo systemctl enable --now checkmk

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
checkmk --version
```

### macOS

```bash
# Using Homebrew
brew install checkmk

# Start service
brew services start checkmk

# Verify installation
checkmk --version
```

### FreeBSD

```bash
# Using pkg
pkg install checkmk

# Enable in rc.conf
echo 'checkmk_enable="YES"' >> /etc/rc.conf

# Start service
service checkmk start

# Verify installation
checkmk --version
```

### Windows

```bash
# Using Chocolatey
choco install checkmk

# Or using Scoop
scoop install checkmk

# Verify installation
checkmk --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/checkmk

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
checkmk --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable checkmk

# Start service
sudo systemctl start checkmk

# Stop service
sudo systemctl stop checkmk

# Restart service
sudo systemctl restart checkmk

# Check status
sudo systemctl status checkmk

# View logs
sudo journalctl -u checkmk -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add checkmk default

# Start service
rc-service checkmk start

# Stop service
rc-service checkmk stop

# Restart service
rc-service checkmk restart

# Check status
rc-service checkmk status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'checkmk_enable="YES"' >> /etc/rc.conf

# Start service
service checkmk start

# Stop service
service checkmk stop

# Restart service
service checkmk restart

# Check status
service checkmk status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start checkmk
brew services stop checkmk
brew services restart checkmk

# Check status
brew services list | grep checkmk
```

### Windows Service Manager

```powershell
# Start service
net start checkmk

# Stop service
net stop checkmk

# Using PowerShell
Start-Service checkmk
Stop-Service checkmk
Restart-Service checkmk

# Check status
Get-Service checkmk
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream checkmk_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name checkmk.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name checkmk.example.com;

    ssl_certificate /etc/ssl/certs/checkmk.example.com.crt;
    ssl_certificate_key /etc/ssl/private/checkmk.example.com.key;

    location / {
        proxy_pass http://checkmk_backend;
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
    ServerName checkmk.example.com
    Redirect permanent / https://checkmk.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName checkmk.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/checkmk.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/checkmk.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend checkmk_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/checkmk.pem
    redirect scheme https if !{ ssl_fc }
    default_backend checkmk_backend

backend checkmk_backend
    balance roundrobin
    server checkmk1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R checkmk:checkmk /etc/checkmk
sudo chmod 750 /etc/checkmk

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
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
sudo systemctl status checkmk

# View logs
sudo journalctl -u checkmk -f

# Monitor resource usage
top -p $(pgrep checkmk)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/checkmk"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/checkmk-backup-$DATE.tar.gz" /etc/checkmk /var/lib/checkmk

echo "Backup completed: $BACKUP_DIR/checkmk-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop checkmk

# Restore from backup
tar -xzf /backup/checkmk/checkmk-backup-*.tar.gz -C /

# Start service
sudo systemctl start checkmk
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u checkmk -n 100
sudo tail -f /var/log/checkmk/checkmk.log

# Check configuration
checkmk --version

# Check permissions
ls -la /etc/checkmk
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80

# Test connectivity
telnet localhost 80

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep checkmk)

# Check disk I/O
iotop -p $(pgrep checkmk)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  checkmk:
    image: checkmk:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/checkmk
      - ./data:/var/lib/checkmk
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update checkmk

# Debian/Ubuntu
sudo apt update && sudo apt upgrade checkmk

# Arch Linux
sudo pacman -Syu checkmk

# Alpine Linux
apk update && apk upgrade checkmk

# openSUSE
sudo zypper update checkmk

# FreeBSD
pkg update && pkg upgrade checkmk

# Always backup before updates
tar -czf /backup/checkmk-pre-update-$(date +%Y%m%d).tar.gz /etc/checkmk

# Restart after updates
sudo systemctl restart checkmk
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/checkmk

# Clean old logs
find /var/log/checkmk -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/checkmk
```

## Additional Resources

- Official Documentation: https://docs.checkmk.org/
- GitHub Repository: https://github.com/checkmk/checkmk
- Community Forum: https://forum.checkmk.org/
- Best Practices Guide: https://docs.checkmk.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
