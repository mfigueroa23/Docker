# Docker Services Repository

[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat&logo=docker&logoColor=white)](https://www.docker.com/)

A curated collection of Docker Compose configurations for deploying various self-hosted services and applications. This repository provides production-ready configurations for workflow automation, VPN access, project management, and file storage solutions.

---

## Table of Contents

- [Overview](#overview)
- [Available Services](#available-services)
- [Prerequisites](#prerequisites)
- [Quick Start Guide](#quick-start-guide)
- [Service Documentation](#service-documentation)
  - [n8n - Workflow Automation](#n8n---workflow-automation)
  - [FossFlow - Open Source Workflow](#fossflow---open-source-workflow)
  - [OpenVPN - VPN Server](#openvpn---vpn-server)
  - [ownCloud - File Storage](#owncloud---file-storage)
  - [Redmine - Project Management](#redmine---project-management)
- [Network Configuration](#network-configuration)
- [Volume Management](#volume-management)
- [Environment Variables](#environment-variables)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)
- [Contributing](#contributing)

---

## Overview

This repository contains Docker Compose configurations for self-hosted services designed for:

- **Workflow Automation**: n8n, FossFlow
- **VPN Access**: OpenVPN Access Server
- **File Storage**: ownCloud
- **Project Management**: Redmine with MySQL

All services are containerized for easy deployment, scalability, and maintenance.

---

## Available Services

| Service | Purpose | Port(s) | Status |
|---------|---------|---------|--------|
| **n8n** | Workflow automation platform | 80, 443, 5678 | Production-ready with SSL |
| **FossFlow** | Open-source workflow management | 8180 | Active |
| **OpenVPN** | VPN server with web UI | 943, 443, 1194/udp | Active |
| **ownCloud** | File sync and share platform | 80 (8080) | Active |
| **Redmine** | Project management tool | 80 (3000), 3306 | Active with MySQL |

---

## Prerequisites

Before deploying any service, ensure you have:

- **Docker Engine** 20.10+ installed
- **Docker Compose** 2.0+ installed
- **Operating System**: Linux, macOS, or Windows with WSL2
- **Ports**: Available ports as listed in the table above
- **Domain** (for n8n): A registered domain with DNS access
- **Storage**: Sufficient disk space for volumes

### Installation Check

```bash
# Verify Docker installation
docker --version
docker compose version

# Check available ports
netstat -tuln | grep -E ':(80|443|943|1194|3000|3306|5678|8180)'
```

---

## Quick Start Guide

### 1. Clone Repository

```bash
git clone <repository-url>
cd Docker
```

### 2. Choose a Service

```bash
# List available services
ls -1 *.yaml

# Output:
# fossflow.yaml
# n8n.yaml
# openvpn.yaml
# ownclod.yaml
# redmine.yaml
```

### 3. Configure Environment Variables

Each service may require environment variables. Create the necessary `.env` files:

```bash
# Example for n8n (already provided)
cat n8n.env

# Example for ownCloud (create if needed)
touch .owncloud.env

# Example for Redmine (create if needed)
touch .redmine.env
touch .mysql.env
```

### 4. Deploy a Service

```bash
# Start service in detached mode
docker compose -f <service>.yaml up -d

# Example: Start n8n
docker compose -f n8n.yaml up -d

# View logs
docker compose -f n8n.yaml logs -f

# Stop service
docker compose -f n8n.yaml down
```

---

## Service Documentation

### n8n - Workflow Automation

**n8n** is a fair-code licensed workflow automation tool that enables you to connect anything to everything via powerful APIs and databases.

#### Architecture

- **Traefik** - Reverse proxy with automatic SSL/TLS certificates
- **n8n** - Workflow automation engine

#### Configuration

**File**: `n8n.yaml`
**Environment File**: `n8n.env`

**Required Environment Variables**:

```bash
# Domain configuration
DOMAIN_NAME=devsonic.cl              # Your top-level domain
SUBDOMAIN=n8n                        # Subdomain for n8n access

# Timezone
GENERIC_TIMEZONE=America/Santiago    # Your timezone

# SSL/TLS
SSL_EMAIL=mfigueroa@devsonic.cl     # Email for Let's Encrypt certificates
```

#### Deployment

```bash
# 1. Verify environment variables
cat n8n.env

# 2. Start services
docker compose -f n8n.yaml up -d

# 3. Check status
docker compose -f n8n.yaml ps

# 4. View logs
docker compose -f n8n.yaml logs -f traefik
docker compose -f n8n.yaml logs -f n8n
```

#### Access

- **Public URL**: `https://n8n.devsonic.cl` (or your configured domain)
- **Local Access**: `http://localhost:5678`
- **Traefik Dashboard**: `http://localhost:8080` (if enabled)

#### Features

- Automatic SSL/TLS certificate provisioning via Let's Encrypt
- HTTP to HTTPS redirection
- Security headers (HSTS, XSS protection, etc.)
- Persistent data storage
- Local file access via `/files` volume mount

#### Volumes

- `n8n_data`: Persistent n8n workflows and credentials
- `traefik_data`: SSL/TLS certificates storage
- `./local-files:/files`: Local file access for workflows

#### Security

- Enforces HTTPS with automatic redirects
- Strict Transport Security (HSTS) enabled
- XSS filter and content type sniffing protection
- File permissions enforcement

---

### FossFlow - Open Source Workflow

**FossFlow** is an open-source workflow management tool for collaborative environments.

#### Configuration

**File**: `fossflow.yaml`

**Environment Variables**:

```yaml
NODE_ENV: production
ENABLE_SERVER_STORAGE: true (default)
ENABLE_GIT_BACKUP: false (default)
```

#### Deployment

```bash
# Start FossFlow
docker compose -f fossflow.yaml up -d

# Check logs
docker compose -f fossflow.yaml logs -f

# Stop service
docker compose -f fossflow.yaml down
```

#### Access

- **URL**: `http://localhost:8180`

#### Features

- Production-ready configuration
- Server-side storage enabled by default
- Optional Git backup support
- Always pulls latest image on restart

---

### OpenVPN - VPN Server

**OpenVPN Access Server** provides secure VPN access with an intuitive web interface.

#### Configuration

**File**: `openvpn.yaml`

**Special Requirements**:
- Requires `NET_ADMIN` capability for network management
- Persistent configuration via volume mount

#### Deployment

```bash
# Start OpenVPN Access Server
docker compose -f openvpn.yaml up -d

# First-time setup (get admin password)
docker logs openvpn | grep "Auto-generated"

# Check status
docker compose -f openvpn.yaml ps
```

#### Access

- **Admin UI**: `https://localhost:943/admin`
- **Client UI**: `https://localhost:943`
- **VPN Protocol**: UDP port 1194

#### Initial Setup

1. Access admin UI at `https://localhost:943/admin`
2. Login with auto-generated credentials (check logs)
3. Configure network settings
4. Create user accounts
5. Download client configuration files

#### Volumes

- `./openvpn:/openvpn`: Persistent OpenVPN configuration and data

#### Features

- Web-based administration interface
- Multi-platform client support
- Persistent configuration storage
- Auto-restart unless manually stopped

---

### ownCloud - File Storage

**ownCloud** is a self-hosted file sync and share platform for secure data management.

#### Configuration

**File**: `ownclod.yaml`
**Environment File**: `.owncloud.env` (required)

**Network Configuration**:
- Custom network: `own_cloud_net` (10.101.1.0/24)
- Static IP: 10.101.1.10

#### Environment File Setup

Create `.owncloud.env` with the following variables:

```bash
# Database configuration
OWNCLOUD_DB_TYPE=mysql
OWNCLOUD_DB_NAME=owncloud
OWNCLOUD_DB_USERNAME=owncloud
OWNCLOUD_DB_PASSWORD=your_secure_password
OWNCLOUD_DB_HOST=mysql_host

# Admin credentials
OWNCLOUD_ADMIN_USERNAME=admin
OWNCLOUD_ADMIN_PASSWORD=your_admin_password

# Domain
OWNCLOUD_DOMAIN=localhost:8080
```

#### Deployment

```bash
# 1. Create environment file
touch .owncloud.env
# Add required variables (see above)

# 2. Create volume directory
sudo mkdir -p /opt/volumes/ownCloud

# 3. Start ownCloud
docker compose -f ownclod.yaml up -d

# 4. Check logs
docker compose -f ownclod.yaml logs -f
```

#### Access

- **URL**: `http://localhost:80` (mapped from container port 8080)
- **Internal IP**: 10.101.1.10

#### Volumes

- `/opt/volumes/ownCloud/:/var/www/html/`: Persistent file storage

#### Features

- ownCloud Server 10
- Persistent file storage
- Custom network isolation
- Always restarts on failure

---

### Redmine - Project Management

**Redmine** is a flexible project management web application with issue tracking, Gantt charts, and wiki functionality.

#### Architecture

- **Redmine**: Web application server
- **MySQL 8.1.0**: Database backend

#### Configuration

**File**: `redmine.yaml`
**Environment Files**:
- `.redmine.env` (required)
- `.mysql.env` (required)

**Network Configuration**:
- Custom network: `redmine_net` (10.101.1.0/24)
- Redmine IP: 10.101.1.10
- MySQL IP: 10.101.1.11

#### Environment File Setup

**`.redmine.env`**:
```bash
# Database connection
REDMINE_DB_MYSQL=10.101.1.11
REDMINE_DB_PORT=3306
REDMINE_DB_DATABASE=redmine
REDMINE_DB_USERNAME=redmine
REDMINE_DB_PASSWORD=your_secure_password

# Optional: Secret key base
REDMINE_SECRET_KEY_BASE=your_random_secret_key
```

**`.mysql.env`**:
```bash
# MySQL root password
MYSQL_ROOT_PASSWORD=your_root_password

# Database for Redmine
MYSQL_DATABASE=redmine
MYSQL_USER=redmine
MYSQL_PASSWORD=your_secure_password
```

#### Deployment

```bash
# 1. Create environment files
touch .redmine.env .mysql.env
# Add required variables (see above)

# 2. Create database volume directory
mkdir -p .redmine_db

# 3. Start Redmine and MySQL
docker compose -f redmine.yaml up -d

# 4. Wait for MySQL initialization (first run)
docker compose -f redmine.yaml logs -f redmine_db

# 5. Check Redmine logs
docker compose -f redmine.yaml logs -f redmine
```

#### Access

- **Redmine UI**: `http://localhost:80` (mapped from container port 3000)
- **MySQL**: `localhost:3306` (if external access needed)
- **Default Credentials**: admin/admin (change immediately!)

#### Volumes

- `./.redmine_db/:/var/lib/mysql/`: Persistent MySQL database

#### Features

- Redmine latest version
- MySQL 8.1.0 database
- Isolated network configuration
- Persistent database storage
- Always restarts on failure

---

## Network Configuration

### Custom Networks

Two services use custom Docker networks for isolation:

#### ownCloud Network

```yaml
Network Name: own_cloud_net
Subnet: 10.101.1.0/24
Gateway: 10.101.1.1 (default)

Containers:
- own_cloud_cl: 10.101.1.10
```

#### Redmine Network

```yaml
Network Name: redmine_net
Subnet: 10.101.1.0/24
Gateway: 10.101.1.1 (default)

Containers:
- redmine: 10.101.1.10
- redmine_db: 10.101.1.11
```

**Note**: Both use the same subnet but different network names, so they are isolated from each other.

### Default Bridge Network

The following services use Docker's default bridge network:
- n8n (managed by Traefik)
- FossFlow
- OpenVPN

---

## Volume Management

### Persistent Volumes

| Service | Volume Type | Mount Point | Purpose |
|---------|-------------|-------------|---------|
| **n8n** | Named | `n8n_data:/home/node/.n8n` | Workflows, credentials |
| **n8n** | Bind | `./local-files:/files` | Local file access |
| **Traefik** | Named | `traefik_data:/letsencrypt` | SSL certificates |
| **OpenVPN** | Bind | `./openvpn:/openvpn` | VPN configuration |
| **ownCloud** | Bind | `/opt/volumes/ownCloud/` | File storage |
| **Redmine** | Bind | `./.redmine_db/` | MySQL database |

### Volume Commands

```bash
# List all volumes
docker volume ls

# Inspect specific volume
docker volume inspect n8n_data

# Backup named volume
docker run --rm -v n8n_data:/data -v $(pwd):/backup alpine \
  tar czf /backup/n8n_backup.tar.gz -C /data .

# Restore named volume
docker run --rm -v n8n_data:/data -v $(pwd):/backup alpine \
  tar xzf /backup/n8n_backup.tar.gz -C /data

# Remove unused volumes
docker volume prune
```

### Backup Recommendations

```bash
# Backup ownCloud files
sudo tar czf owncloud_backup.tar.gz -C /opt/volumes ownCloud

# Backup Redmine database
docker exec redmine_db mysqldump -u redmine -p redmine > redmine_backup.sql

# Backup OpenVPN configuration
tar czf openvpn_backup.tar.gz openvpn/
```

---

## Environment Variables

### n8n Service

| Variable | Description | Example |
|----------|-------------|---------|
| `DOMAIN_NAME` | Top-level domain | `devsonic.cl` |
| `SUBDOMAIN` | Subdomain for n8n | `n8n` |
| `GENERIC_TIMEZONE` | Timezone for scheduling | `America/Santiago` |
| `SSL_EMAIL` | Email for Let's Encrypt | `admin@example.com` |

### FossFlow Service

| Variable | Description | Default |
|----------|-------------|---------|
| `NODE_ENV` | Node environment | `production` |
| `ENABLE_SERVER_STORAGE` | Enable server storage | `true` |
| `ENABLE_GIT_BACKUP` | Enable Git backups | `false` |

### ownCloud Service

Required variables in `.owncloud.env`:
- Database configuration (type, name, username, password, host)
- Admin credentials
- Domain name

### Redmine Service

Required variables in `.redmine.env`:
- Database connection details
- Optional secret key base

Required variables in `.mysql.env`:
- Root password
- Database name and user credentials

---

## Troubleshooting

### Common Issues

#### Port Conflicts

```bash
# Check if port is already in use
sudo lsof -i :80
sudo lsof -i :443

# Stop conflicting service
sudo systemctl stop nginx  # or apache2

# Or change port mapping in docker-compose file
ports:
  - "8080:80"  # Use port 8080 instead of 80
```

#### Container Won't Start

```bash
# Check logs
docker compose -f <service>.yaml logs

# Inspect container
docker inspect <container_name>

# Verify environment files exist
ls -la .*.env

# Check file permissions
ls -la /opt/volumes/ownCloud
```

#### n8n SSL Certificate Issues

```bash
# Check Traefik logs
docker compose -f n8n.yaml logs traefik

# Verify DNS points to your server
nslookup n8n.devsonic.cl

# Check certificate storage
docker volume inspect traefik_data

# Manually check Let's Encrypt rate limits
# https://letsencrypt.org/docs/rate-limits/
```

#### Database Connection Errors (Redmine/ownCloud)

```bash
# Check if database is ready
docker compose -f redmine.yaml logs redmine_db

# Test database connection
docker exec -it redmine_db mysql -u redmine -p

# Verify network connectivity
docker network inspect redmine_net

# Check environment variables are loaded
docker exec redmine env | grep REDMINE_DB
```

#### OpenVPN Admin Password

```bash
# Retrieve auto-generated password
docker logs openvpn 2>&1 | grep -i password

# Or set custom password
docker exec -it openvpn passwd openvpn
```

### Logs and Debugging

```bash
# View real-time logs
docker compose -f <service>.yaml logs -f

# View last 100 lines
docker compose -f <service>.yaml logs --tail 100

# Save logs to file
docker compose -f <service>.yaml logs > debug.log

# Check container resource usage
docker stats

# Inspect container details
docker inspect <container_name>
```

### Reset Service

```bash
# Stop and remove containers
docker compose -f <service>.yaml down

# Remove volumes (CAUTION: deletes all data)
docker compose -f <service>.yaml down -v

# Remove specific volume
docker volume rm <volume_name>

# Start fresh
docker compose -f <service>.yaml up -d
```

---

## Best Practices

### Security

1. **Change Default Passwords**
   - Redmine default: admin/admin
   - OpenVPN auto-generated password
   - Database passwords

2. **Use Strong Passwords**
   ```bash
   # Generate secure password
   openssl rand -base64 32
   ```

3. **Protect Environment Files**
   ```bash
   chmod 600 .*.env
   ```

4. **Regular Updates**
   ```bash
   # Pull latest images
   docker compose -f <service>.yaml pull

   # Recreate containers with new images
   docker compose -f <service>.yaml up -d --force-recreate
   ```

5. **Enable Firewall**
   ```bash
   # Example: UFW on Ubuntu
   sudo ufw allow 80/tcp
   sudo ufw allow 443/tcp
   sudo ufw allow 1194/udp
   ```

### Backup Strategy

1. **Regular Backups**
   - Schedule daily backups with cron
   - Store backups off-site
   - Test restore procedures

2. **Version Control**
   - Keep docker-compose files in Git
   - Don't commit `.env` files (use `.env.example`)
   - Document configuration changes

### Monitoring

1. **Health Checks**
   ```bash
   # Check service status
   docker compose ps

   # Monitor resource usage
   docker stats
   ```

2. **Log Rotation**
   ```bash
   # Configure Docker daemon logging
   cat /etc/docker/daemon.json
   {
     "log-driver": "json-file",
     "log-opts": {
       "max-size": "10m",
       "max-file": "3"
     }
   }
   ```

### Performance

1. **Resource Limits**
   ```yaml
   # Add to service definition
   deploy:
     resources:
       limits:
         cpus: '0.5'
         memory: 512M
       reservations:
         cpus: '0.25'
         memory: 256M
   ```

2. **Volume Performance**
   - Use named volumes for databases
   - Consider SSD for I/O intensive services
   - Regular cleanup of unused volumes

---

## Contributing

Contributions are welcome! Please follow these guidelines:

1. **Fork the Repository**
2. **Create Feature Branch**
   ```bash
   git checkout -b feature/new-service
   ```
3. **Test Configuration**
   ```bash
   docker compose -f <new-service>.yaml config
   docker compose -f <new-service>.yaml up -d
   ```
4. **Document Changes**
   - Update README.md
   - Add environment file examples
   - Include troubleshooting notes
5. **Submit Pull Request**

### Adding New Services

When adding a new service, include:

- Docker Compose YAML file
- Environment file example (`.env.example`)
- Documentation section in README
- Network configuration if needed
- Volume requirements
- Security considerations

---

## License

This repository is provided as-is for educational and production use. Individual services are subject to their respective licenses:

- **n8n**: Fair-code license (Sustainable Use License)
- **FossFlow**: Open source
- **OpenVPN**: Access Server license
- **ownCloud**: AGPLv3
- **Redmine**: GPLv2

---

## Support

For service-specific issues:
- **n8n**: https://community.n8n.io
- **FossFlow**: Check project repository
- **OpenVPN**: https://openvpn.net/community
- **ownCloud**: https://central.owncloud.org
- **Redmine**: https://www.redmine.org/projects/redmine/wiki

For configuration issues, please open an issue in this repository.

---

## Acknowledgments

Thanks to the open-source community and the developers of:
- Docker and Docker Compose
- Traefik reverse proxy
- n8n workflow automation
- ownCloud file sharing
- Redmine project management
- OpenVPN secure VPN

---

**Last Updated**: 2026-02-13
**Maintained By**: DevSonic Team
**Repository**: /Users/sysadmin/Workspace/Docker
