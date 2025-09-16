# isc-dhcp Installation Guide

isc-dhcp is a free and open-source DHCP server and client. ISC DHCP provides robust DHCP services for automatic network configuration, the reference implementation of DHCP

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
  - Storage: 50MB for installation
  - Network: UDP ports 67-68
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 67 (default isc-dhcp port)
  - Port 68 for client
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

# Install isc-dhcp
sudo dnf install -y isc-dhcp-server

# Enable and start service
sudo systemctl enable --now isc-dhcp-server

# Configure firewall
sudo firewall-cmd --permanent --add-port=67/tcp
sudo firewall-cmd --reload

# Verify installation
dhcpd --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install isc-dhcp
sudo apt install -y isc-dhcp-server

# Enable and start service
sudo systemctl enable --now isc-dhcp-server

# Configure firewall
sudo ufw allow 67

# Verify installation
dhcpd --version
```

### Arch Linux

```bash
# Install isc-dhcp
sudo pacman -S isc-dhcp-server

# Enable and start service
sudo systemctl enable --now isc-dhcp-server

# Verify installation
dhcpd --version
```

### Alpine Linux

```bash
# Install isc-dhcp
apk add --no-cache isc-dhcp-server

# Enable and start service
rc-update add isc-dhcp-server default
rc-service isc-dhcp-server start

# Verify installation
dhcpd --version
```

### openSUSE/SLES

```bash
# Install isc-dhcp
sudo zypper install -y isc-dhcp-server

# Enable and start service
sudo systemctl enable --now isc-dhcp-server

# Configure firewall
sudo firewall-cmd --permanent --add-port=67/tcp
sudo firewall-cmd --reload

# Verify installation
dhcpd --version
```

### macOS

```bash
# Using Homebrew
brew install isc-dhcp-server

# Start service
brew services start isc-dhcp-server

# Verify installation
dhcpd --version
```

### FreeBSD

```bash
# Using pkg
pkg install isc-dhcp-server

# Enable in rc.conf
echo 'isc-dhcp-server_enable="YES"' >> /etc/rc.conf

# Start service
service isc-dhcp-server start

# Verify installation
dhcpd --version
```

### Windows

```bash
# Using Chocolatey
choco install isc-dhcp-server

# Or using Scoop
scoop install isc-dhcp-server

# Verify installation
dhcpd --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/isc-dhcp-server

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
dhcpd --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable isc-dhcp-server

# Start service
sudo systemctl start isc-dhcp-server

# Stop service
sudo systemctl stop isc-dhcp-server

# Restart service
sudo systemctl restart isc-dhcp-server

# Check status
sudo systemctl status isc-dhcp-server

# View logs
sudo journalctl -u isc-dhcp-server -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add isc-dhcp-server default

# Start service
rc-service isc-dhcp-server start

# Stop service
rc-service isc-dhcp-server stop

# Restart service
rc-service isc-dhcp-server restart

# Check status
rc-service isc-dhcp-server status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'isc-dhcp-server_enable="YES"' >> /etc/rc.conf

# Start service
service isc-dhcp-server start

# Stop service
service isc-dhcp-server stop

# Restart service
service isc-dhcp-server restart

# Check status
service isc-dhcp-server status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start isc-dhcp-server
brew services stop isc-dhcp-server
brew services restart isc-dhcp-server

# Check status
brew services list | grep isc-dhcp-server
```

### Windows Service Manager

```powershell
# Start service
net start isc-dhcp-server

# Stop service
net stop isc-dhcp-server

# Using PowerShell
Start-Service isc-dhcp-server
Stop-Service isc-dhcp-server
Restart-Service isc-dhcp-server

# Check status
Get-Service isc-dhcp-server
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream isc-dhcp-server_backend {
    server 127.0.0.1:67;
}

server {
    listen 80;
    server_name isc-dhcp-server.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name isc-dhcp-server.example.com;

    ssl_certificate /etc/ssl/certs/isc-dhcp-server.example.com.crt;
    ssl_certificate_key /etc/ssl/private/isc-dhcp-server.example.com.key;

    location / {
        proxy_pass http://isc-dhcp-server_backend;
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
    ServerName isc-dhcp-server.example.com
    Redirect permanent / https://isc-dhcp-server.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName isc-dhcp-server.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/isc-dhcp-server.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/isc-dhcp-server.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:67/
    ProxyPassReverse / http://127.0.0.1:67/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend isc-dhcp-server_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/isc-dhcp-server.pem
    redirect scheme https if !{ ssl_fc }
    default_backend isc-dhcp-server_backend

backend isc-dhcp-server_backend
    balance roundrobin
    server isc-dhcp-server1 127.0.0.1:67 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R isc-dhcp-server:isc-dhcp-server /etc/isc-dhcp-server
sudo chmod 750 /etc/isc-dhcp-server

# Configure firewall
sudo firewall-cmd --permanent --add-port=67/tcp
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
sudo systemctl status isc-dhcp-server

# View logs
sudo journalctl -u isc-dhcp-server -f

# Monitor resource usage
top -p $(pgrep isc-dhcp-server)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/isc-dhcp-server"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/isc-dhcp-server-backup-$DATE.tar.gz" /etc/isc-dhcp-server /var/lib/isc-dhcp-server

echo "Backup completed: $BACKUP_DIR/isc-dhcp-server-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop isc-dhcp-server

# Restore from backup
tar -xzf /backup/isc-dhcp-server/isc-dhcp-server-backup-*.tar.gz -C /

# Start service
sudo systemctl start isc-dhcp-server
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u isc-dhcp-server -n 100
sudo tail -f /var/log/isc-dhcp-server/isc-dhcp-server.log

# Check configuration
dhcpd --version

# Check permissions
ls -la /etc/isc-dhcp-server
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 67

# Test connectivity
telnet localhost 67

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep isc-dhcp-server)

# Check disk I/O
iotop -p $(pgrep isc-dhcp-server)

# Check connections
ss -an | grep 67
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  isc-dhcp-server:
    image: isc-dhcp-server:latest
    ports:
      - "67:67"
    volumes:
      - ./config:/etc/isc-dhcp-server
      - ./data:/var/lib/isc-dhcp-server
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update isc-dhcp-server

# Debian/Ubuntu
sudo apt update && sudo apt upgrade isc-dhcp-server

# Arch Linux
sudo pacman -Syu isc-dhcp-server

# Alpine Linux
apk update && apk upgrade isc-dhcp-server

# openSUSE
sudo zypper update isc-dhcp-server

# FreeBSD
pkg update && pkg upgrade isc-dhcp-server

# Always backup before updates
tar -czf /backup/isc-dhcp-server-pre-update-$(date +%Y%m%d).tar.gz /etc/isc-dhcp-server

# Restart after updates
sudo systemctl restart isc-dhcp-server
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/isc-dhcp-server

# Clean old logs
find /var/log/isc-dhcp-server -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/isc-dhcp-server
```

## Additional Resources

- Official Documentation: https://docs.isc-dhcp-server.org/
- GitHub Repository: https://github.com/isc-dhcp-server/isc-dhcp-server
- Community Forum: https://forum.isc-dhcp-server.org/
- Best Practices Guide: https://docs.isc-dhcp-server.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
