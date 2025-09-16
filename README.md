# mediawiki Installation Guide

mediawiki is a free and open-source wiki platform. MediaWiki powers Wikipedia and provides collaborative documentation platform

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
  - RAM: 512MB minimum
  - Storage: 1GB for data
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default mediawiki port)
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

# Install mediawiki
sudo dnf install -y mediawiki

# Enable and start service
sudo systemctl enable --now mediawiki

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
mediawiki --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install mediawiki
sudo apt install -y mediawiki

# Enable and start service
sudo systemctl enable --now mediawiki

# Configure firewall
sudo ufw allow 80

# Verify installation
mediawiki --version
```

### Arch Linux

```bash
# Install mediawiki
sudo pacman -S mediawiki

# Enable and start service
sudo systemctl enable --now mediawiki

# Verify installation
mediawiki --version
```

### Alpine Linux

```bash
# Install mediawiki
apk add --no-cache mediawiki

# Enable and start service
rc-update add mediawiki default
rc-service mediawiki start

# Verify installation
mediawiki --version
```

### openSUSE/SLES

```bash
# Install mediawiki
sudo zypper install -y mediawiki

# Enable and start service
sudo systemctl enable --now mediawiki

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
mediawiki --version
```

### macOS

```bash
# Using Homebrew
brew install mediawiki

# Start service
brew services start mediawiki

# Verify installation
mediawiki --version
```

### FreeBSD

```bash
# Using pkg
pkg install mediawiki

# Enable in rc.conf
echo 'mediawiki_enable="YES"' >> /etc/rc.conf

# Start service
service mediawiki start

# Verify installation
mediawiki --version
```

### Windows

```bash
# Using Chocolatey
choco install mediawiki

# Or using Scoop
scoop install mediawiki

# Verify installation
mediawiki --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/mediawiki

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
mediawiki --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable mediawiki

# Start service
sudo systemctl start mediawiki

# Stop service
sudo systemctl stop mediawiki

# Restart service
sudo systemctl restart mediawiki

# Check status
sudo systemctl status mediawiki

# View logs
sudo journalctl -u mediawiki -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add mediawiki default

# Start service
rc-service mediawiki start

# Stop service
rc-service mediawiki stop

# Restart service
rc-service mediawiki restart

# Check status
rc-service mediawiki status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'mediawiki_enable="YES"' >> /etc/rc.conf

# Start service
service mediawiki start

# Stop service
service mediawiki stop

# Restart service
service mediawiki restart

# Check status
service mediawiki status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start mediawiki
brew services stop mediawiki
brew services restart mediawiki

# Check status
brew services list | grep mediawiki
```

### Windows Service Manager

```powershell
# Start service
net start mediawiki

# Stop service
net stop mediawiki

# Using PowerShell
Start-Service mediawiki
Stop-Service mediawiki
Restart-Service mediawiki

# Check status
Get-Service mediawiki
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream mediawiki_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name mediawiki.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name mediawiki.example.com;

    ssl_certificate /etc/ssl/certs/mediawiki.example.com.crt;
    ssl_certificate_key /etc/ssl/private/mediawiki.example.com.key;

    location / {
        proxy_pass http://mediawiki_backend;
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
    ServerName mediawiki.example.com
    Redirect permanent / https://mediawiki.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName mediawiki.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/mediawiki.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/mediawiki.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend mediawiki_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/mediawiki.pem
    redirect scheme https if !{ ssl_fc }
    default_backend mediawiki_backend

backend mediawiki_backend
    balance roundrobin
    server mediawiki1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R mediawiki:mediawiki /etc/mediawiki
sudo chmod 750 /etc/mediawiki

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
sudo systemctl status mediawiki

# View logs
sudo journalctl -u mediawiki -f

# Monitor resource usage
top -p $(pgrep mediawiki)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/mediawiki"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/mediawiki-backup-$DATE.tar.gz" /etc/mediawiki /var/lib/mediawiki

echo "Backup completed: $BACKUP_DIR/mediawiki-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop mediawiki

# Restore from backup
tar -xzf /backup/mediawiki/mediawiki-backup-*.tar.gz -C /

# Start service
sudo systemctl start mediawiki
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u mediawiki -n 100
sudo tail -f /var/log/mediawiki/mediawiki.log

# Check configuration
mediawiki --version

# Check permissions
ls -la /etc/mediawiki
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
top -p $(pgrep mediawiki)

# Check disk I/O
iotop -p $(pgrep mediawiki)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  mediawiki:
    image: mediawiki:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/mediawiki
      - ./data:/var/lib/mediawiki
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update mediawiki

# Debian/Ubuntu
sudo apt update && sudo apt upgrade mediawiki

# Arch Linux
sudo pacman -Syu mediawiki

# Alpine Linux
apk update && apk upgrade mediawiki

# openSUSE
sudo zypper update mediawiki

# FreeBSD
pkg update && pkg upgrade mediawiki

# Always backup before updates
tar -czf /backup/mediawiki-pre-update-$(date +%Y%m%d).tar.gz /etc/mediawiki

# Restart after updates
sudo systemctl restart mediawiki
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/mediawiki

# Clean old logs
find /var/log/mediawiki -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/mediawiki
```

## Additional Resources

- Official Documentation: https://docs.mediawiki.org/
- GitHub Repository: https://github.com/mediawiki/mediawiki
- Community Forum: https://forum.mediawiki.org/
- Best Practices Guide: https://docs.mediawiki.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
