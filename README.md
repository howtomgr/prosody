# Prosody Installation Guide

Prosody is a free and open-source XMPP Server. Modern XMPP communication server

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
  - CPU: 2 cores minimum (4+ cores recommended)
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 1GB for installation
  - Network: 5222/5269 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 5222/5269 (default prosody port)
- **Dependencies**:
  - lua5.2, lua-socket, lua-expat
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

# Install prosody
sudo dnf install -y prosody lua5.2, lua-socket, lua-expat

# Enable and start service
sudo systemctl enable --now prosody

# Configure firewall
sudo firewall-cmd --permanent --add-service=prosody
sudo firewall-cmd --reload

# Verify installation
prosody --version || systemctl status prosody
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install prosody
sudo apt install -y prosody lua5.2, lua-socket, lua-expat

# Enable and start service
sudo systemctl enable --now prosody

# Configure firewall
sudo ufw allow 5222/5269

# Verify installation
prosody --version || systemctl status prosody
```

### Arch Linux

```bash
# Install prosody
sudo pacman -S prosody

# Enable and start service
sudo systemctl enable --now prosody

# Verify installation
prosody --version || systemctl status prosody
```

### Alpine Linux

```bash
# Install prosody
apk add --no-cache prosody

# Enable and start service
rc-update add prosody default
rc-service prosody start

# Verify installation
prosody --version || rc-service prosody status
```

### openSUSE/SLES

```bash
# Install prosody
sudo zypper install -y prosody lua5.2, lua-socket, lua-expat

# Enable and start service
sudo systemctl enable --now prosody

# Configure firewall
sudo firewall-cmd --permanent --add-service=prosody
sudo firewall-cmd --reload

# Verify installation
prosody --version || systemctl status prosody
```

### macOS

```bash
# Using Homebrew
brew install prosody

# Start service
brew services start prosody

# Verify installation
prosody --version
```

### FreeBSD

```bash
# Using pkg
pkg install prosody

# Enable in rc.conf
echo 'prosody_enable="YES"' >> /etc/rc.conf

# Start service
service prosody start

# Verify installation
prosody --version || service prosody status
```

### Windows

```powershell
# Using Chocolatey
choco install prosody

# Or using Scoop
scoop install prosody

# Verify installation
prosody --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /etc/prosody

# Set up basic configuration
sudo tee /etc/prosody/prosody.conf << 'EOF'
# Prosody Configuration
c2s_tcp_keepalives = true
EOF

# Test configuration
sudo prosody -t || sudo prosody configtest

# Reload service
sudo systemctl reload prosody
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R prosody:prosody /etc/prosody
sudo chmod 750 /etc/prosody

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable prosody

# Start service
sudo systemctl start prosody

# Stop service
sudo systemctl stop prosody

# Restart service
sudo systemctl restart prosody

# Reload configuration
sudo systemctl reload prosody

# Check status
sudo systemctl status prosody

# View logs
sudo journalctl -u prosody -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add prosody default

# Start service
rc-service prosody start

# Stop service
rc-service prosody stop

# Restart service
rc-service prosody restart

# Check status
rc-service prosody status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'prosody_enable="YES"' >> /etc/rc.conf

# Start service
service prosody start

# Stop service
service prosody stop

# Restart service
service prosody restart

# Check status
service prosody status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start prosody
brew services stop prosody
brew services restart prosody

# Check status
brew services list | grep prosody
```

### Windows Service Manager

```powershell
# Start service
net start prosody

# Stop service
net stop prosody

# Using PowerShell
Start-Service prosody
Stop-Service prosody
Restart-Service prosody

# Check status
Get-Service prosody
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /etc/prosody/prosody.conf << 'EOF'
c2s_tcp_keepalives = true
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart prosody
```

### Clustering and High Availability

```bash
# Configure clustering (if supported)
# See official documentation for cluster setup

# Basic load balancing setup example
# Configure multiple instances on different ports
```

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream prosody_backend {
    server 127.0.0.1:5222/5269;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name prosody.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name prosody.example.com;

    ssl_certificate /etc/ssl/certs/prosody.example.com.crt;
    ssl_certificate_key /etc/ssl/private/prosody.example.com.key;

    location / {
        proxy_pass http://prosody_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support (if needed)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName prosody.example.com
    Redirect permanent / https://prosody.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName prosody.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/prosody.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/prosody.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:5222/5269/
    ProxyPassReverse / http://127.0.0.1:5222/5269/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:5222/5269/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend prosody_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/prosody.pem
    redirect scheme https if !{ ssl_fc }
    default_backend prosody_backend

backend prosody_backend
    balance roundrobin
    option httpchk GET /health
    server prosody1 127.0.0.1:5222/5269 check
    server prosody2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R prosody:prosody /etc/prosody
sudo chmod 750 /etc/prosody

# Configure firewall
sudo firewall-cmd --permanent --add-service=prosody
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/prosody.conf << 'EOF'
[prosody]
enabled = true
port = 5222/5269
filter = prosody
logpath = /var/log/prosody/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/prosody.key \
    -out /etc/ssl/certs/prosody.crt

# Configure SSL in prosody
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE prosody_db;
CREATE USER prosody_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE prosody_db TO prosody_user;
EOF

# Configure prosody to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE prosody_db;
CREATE USER 'prosody_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON prosody_db.* TO 'prosody_user'@'localhost';
FLUSH PRIVILEGES;
EOF
```

## Performance Optimization

### System Tuning

```bash
# Kernel parameters
sudo tee -a /etc/sysctl.conf << EOF
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
net.core.netdev_max_backlog = 5000
vm.swappiness = 10
EOF

sudo sysctl -p

# Prosody specific tuning
c2s_tcp_keepalives = true
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
prosody soft nofile 65535
prosody hard nofile 65535
prosody soft nproc 32768
prosody hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'prosody'
    static_configs:
      - targets: ['localhost:5222/5269']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet prosody; then
    echo "Prosody is running"
    exit 0
else
    echo "Prosody is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/prosody << 'EOF'
/var/log/prosody/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 prosody prosody
    postrotate
        systemctl reload prosody > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Prosody backup script
BACKUP_DIR="/backup/prosody"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop prosody

# Backup configuration
tar -czf "$BACKUP_DIR/prosody-config-$DATE.tar.gz" /etc/prosody

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/prosody-data-$DATE.tar.gz" /var/lib/prosody

# Start service
systemctl start prosody

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop prosody

# Restore configuration
sudo tar -xzf /backup/prosody/prosody-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/prosody/prosody-data-*.tar.gz -C /

# Set permissions
sudo chown -R prosody:prosody /etc/prosody
sudo chown -R prosody:prosody /var/lib/prosody

# Start service
sudo systemctl start prosody
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u prosody -n 100
sudo tail -f /var/log/prosody/*.log

# Check configuration
sudo prosody -t || sudo prosody configtest

# Check permissions
ls -la /etc/prosody
ls -la /var/lib/prosody
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 5222/5269
sudo netstat -tlnp | grep 5222/5269

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 5222/5269
nc -zv localhost 5222/5269
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep lua5.2)
htop -p $(pgrep lua5.2)

# Check connections
ss -ant | grep :5222/5269 | wc -l

# Monitor I/O
iotop -p $(pgrep lua5.2)
```

### Debug Mode

```bash
# Run in debug mode
sudo prosody -d
# or
sudo prosody debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  prosody:
    image: prosody:latest
    container_name: prosody
    ports:
      - "5222/5269:5222/5269"
    volumes:
      - ./config:/etc/prosody
      - ./data:/var/lib/prosody
    environment:
      - prosody_CONFIG=/etc/prosody/prosody.conf
    restart: unless-stopped
    networks:
      - prosody_net

networks:
  prosody_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prosody
spec:
  replicas: 3
  selector:
    matchLabels:
      app: prosody
  template:
    metadata:
      labels:
        app: prosody
    spec:
      containers:
      - name: prosody
        image: prosody:latest
        ports:
        - containerPort: 5222/5269
        volumeMounts:
        - name: config
          mountPath: /etc/prosody
      volumes:
      - name: config
        configMap:
          name: prosody-config
---
apiVersion: v1
kind: Service
metadata:
  name: prosody
spec:
  selector:
    app: prosody
  ports:
  - port: 5222/5269
    targetPort: 5222/5269
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure Prosody
  hosts: all
  become: yes
  tasks:
    - name: Install prosody
      package:
        name: prosody
        state: present
    
    - name: Configure prosody
      template:
        src: prosody.conf.j2
        dest: /etc/prosody/prosody.conf
        owner: prosody
        group: prosody
        mode: '0640'
      notify: restart prosody
    
    - name: Start and enable prosody
      systemd:
        name: prosody
        state: started
        enabled: yes
  
  handlers:
    - name: restart prosody
      systemd:
        name: prosody
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update prosody

# Debian/Ubuntu
sudo apt update && sudo apt upgrade prosody

# Arch Linux
sudo pacman -Syu prosody

# Alpine Linux
apk update && apk upgrade prosody

# openSUSE
sudo zypper update prosody

# FreeBSD
pkg update && pkg upgrade prosody

# Always backup before updates
tar -czf /backup/prosody-pre-update-$(date +%Y%m%d).tar.gz /etc/prosody

# Restart after updates
sudo systemctl restart prosody
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/prosody -name "*.log" -mtime +30 -delete

# Verify integrity
sudo prosody --verify || sudo prosody check

# Update databases (if applicable)
sudo prosody-update-db

# Optimize performance
sudo prosody-optimize

# Check for security updates
sudo prosody --security-check
```

## Additional Resources

- Official Documentation: https://docs.prosody.org/
- GitHub Repository: https://github.com/prosody/prosody
- Community Forum: https://forum.prosody.org/
- Wiki: https://wiki.prosody.org/
- Comparison vs ejabberd, Openfire, Tigase, Metronome: https://docs.prosody.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
