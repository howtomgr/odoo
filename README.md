# odoo Installation Guide

odoo is a free and open-source business management software. Odoo provides all-in-one business software including CRM, eCommerce, accounting

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
  - RAM: 2GB minimum
  - Storage: 10GB for data
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8069 (default odoo port)
  - Longpolling on 8072
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

# Install odoo
sudo dnf install -y odoo

# Enable and start service
sudo systemctl enable --now odoo

# Configure firewall
sudo firewall-cmd --permanent --add-port=8069/tcp
sudo firewall-cmd --reload

# Verify installation
odoo --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install odoo
sudo apt install -y odoo

# Enable and start service
sudo systemctl enable --now odoo

# Configure firewall
sudo ufw allow 8069

# Verify installation
odoo --version
```

### Arch Linux

```bash
# Install odoo
sudo pacman -S odoo

# Enable and start service
sudo systemctl enable --now odoo

# Verify installation
odoo --version
```

### Alpine Linux

```bash
# Install odoo
apk add --no-cache odoo

# Enable and start service
rc-update add odoo default
rc-service odoo start

# Verify installation
odoo --version
```

### openSUSE/SLES

```bash
# Install odoo
sudo zypper install -y odoo

# Enable and start service
sudo systemctl enable --now odoo

# Configure firewall
sudo firewall-cmd --permanent --add-port=8069/tcp
sudo firewall-cmd --reload

# Verify installation
odoo --version
```

### macOS

```bash
# Using Homebrew
brew install odoo

# Start service
brew services start odoo

# Verify installation
odoo --version
```

### FreeBSD

```bash
# Using pkg
pkg install odoo

# Enable in rc.conf
echo 'odoo_enable="YES"' >> /etc/rc.conf

# Start service
service odoo start

# Verify installation
odoo --version
```

### Windows

```bash
# Using Chocolatey
choco install odoo

# Or using Scoop
scoop install odoo

# Verify installation
odoo --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/odoo

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
odoo --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable odoo

# Start service
sudo systemctl start odoo

# Stop service
sudo systemctl stop odoo

# Restart service
sudo systemctl restart odoo

# Check status
sudo systemctl status odoo

# View logs
sudo journalctl -u odoo -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add odoo default

# Start service
rc-service odoo start

# Stop service
rc-service odoo stop

# Restart service
rc-service odoo restart

# Check status
rc-service odoo status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'odoo_enable="YES"' >> /etc/rc.conf

# Start service
service odoo start

# Stop service
service odoo stop

# Restart service
service odoo restart

# Check status
service odoo status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start odoo
brew services stop odoo
brew services restart odoo

# Check status
brew services list | grep odoo
```

### Windows Service Manager

```powershell
# Start service
net start odoo

# Stop service
net stop odoo

# Using PowerShell
Start-Service odoo
Stop-Service odoo
Restart-Service odoo

# Check status
Get-Service odoo
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream odoo_backend {
    server 127.0.0.1:8069;
}

server {
    listen 80;
    server_name odoo.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name odoo.example.com;

    ssl_certificate /etc/ssl/certs/odoo.example.com.crt;
    ssl_certificate_key /etc/ssl/private/odoo.example.com.key;

    location / {
        proxy_pass http://odoo_backend;
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
    ServerName odoo.example.com
    Redirect permanent / https://odoo.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName odoo.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/odoo.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/odoo.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8069/
    ProxyPassReverse / http://127.0.0.1:8069/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend odoo_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/odoo.pem
    redirect scheme https if !{ ssl_fc }
    default_backend odoo_backend

backend odoo_backend
    balance roundrobin
    server odoo1 127.0.0.1:8069 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R odoo:odoo /etc/odoo
sudo chmod 750 /etc/odoo

# Configure firewall
sudo firewall-cmd --permanent --add-port=8069/tcp
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
sudo systemctl status odoo

# View logs
sudo journalctl -u odoo -f

# Monitor resource usage
top -p $(pgrep odoo)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/odoo"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/odoo-backup-$DATE.tar.gz" /etc/odoo /var/lib/odoo

echo "Backup completed: $BACKUP_DIR/odoo-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop odoo

# Restore from backup
tar -xzf /backup/odoo/odoo-backup-*.tar.gz -C /

# Start service
sudo systemctl start odoo
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u odoo -n 100
sudo tail -f /var/log/odoo/odoo.log

# Check configuration
odoo --version

# Check permissions
ls -la /etc/odoo
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8069

# Test connectivity
telnet localhost 8069

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep odoo)

# Check disk I/O
iotop -p $(pgrep odoo)

# Check connections
ss -an | grep 8069
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  odoo:
    image: odoo:latest
    ports:
      - "8069:8069"
    volumes:
      - ./config:/etc/odoo
      - ./data:/var/lib/odoo
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update odoo

# Debian/Ubuntu
sudo apt update && sudo apt upgrade odoo

# Arch Linux
sudo pacman -Syu odoo

# Alpine Linux
apk update && apk upgrade odoo

# openSUSE
sudo zypper update odoo

# FreeBSD
pkg update && pkg upgrade odoo

# Always backup before updates
tar -czf /backup/odoo-pre-update-$(date +%Y%m%d).tar.gz /etc/odoo

# Restart after updates
sudo systemctl restart odoo
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/odoo

# Clean old logs
find /var/log/odoo -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/odoo
```

## Additional Resources

- Official Documentation: https://docs.odoo.org/
- GitHub Repository: https://github.com/odoo/odoo
- Community Forum: https://forum.odoo.org/
- Best Practices Guide: https://docs.odoo.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
