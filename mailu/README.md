# Mailu Email System Setup Guide

## Introduction

This guide provides a comprehensive walkthrough for setting up a secure, production-ready Mailu email server on Ubuntu 22.04 using Docker and Docker Compose. Mailu is a simple yet full-featured mail server as a set of Docker images.

This setup requires no control panel and includes DNS, SSL, and automated backups.

## Table of Contents

1.  [Prerequisites](#prerequisites)
2.  [Update Ubuntu](#update-ubuntu)
3.  [DNS Settings (Cloudflare)](#dns-settings-cloudflare)
4.  [Install Docker & Docker Compose](#install-docker--docker-compose)
5.  [Mailu Setup & Configuration](#mailu-setup--configuration)
6.  [Start Mailu](#start-mailu)
7.  [DNS Records for Email](#dns-records-for-email)
8.  [First Login & Admin Setup](#first-login--admin-setup)
9.  [Backup System](#backup-system)
10. [Troubleshooting](#troubleshooting)

## Prerequisites

### Requirements

*   Contabo VPS with Ubuntu 22.04 (fresh install, no panel)
*   Root SSH access
*   Domain name (e.g., yourdomain.com)
*   Cloudflare (or other DNS provider)
*   Docker
*   Docker Compose

## Update Ubuntu

```bash
sudo apt update && sudo apt upgrade -y && sudo apt dist-upgrade -y
sudo apt autoremove -y
sudo update-grub
sudo snap refresh
```

(Optional, for testing only): Disable firewall (for initial setup, re-enable and configure later!)

```bash
sudo ufw disable
```

## DNS Settings (Cloudflare)

Go to your domain in Cloudflare. Add the following A record (DNS only, not proxied):

| Type | Name    | Content (IP) | TTL  |
| ---- | ------- | ------------ | ---- |
| A    | webmail | YOUR\_VPS\_IP  | Auto |

Do not enable Cloudflare proxy (orange cloud) for mail services.

## Install Docker & Docker Compose

### Install Docker:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker ${USER}
```

# Log out and log back in to activate docker group

### Install Docker Compose:

```bash
DOCKER_COMPOSE_VERSION=2.29.2
sudo curl -L "https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-$(uname -s)-$(uname -m)" \
  -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

## Mailu Setup & Configuration

### Generate configuration:

Go to [Mailu Setup Wizard](https://setup.mailu.io)

Fill in:

*   Mailu storage path: `/mailu`
*   Main mail domain: `yourdomain.com`
*   Postmaster: `admin`
*   TLS: `letsencrypt`
*   Website name: `webmail.yourdomain.com`
*   Enable admin UI, API, and desired features
*   IPv4 listen address: `YOUR_VPS_IP`
*   Docker subnet: e.g., `192.168.203.0/24`
*   Public hostnames: `webmail.yourdomain.com`

### Download configuration:

```bash
mkdir /mailu
cd /mailu
wget https://setup.mailu.io/2024.06/file/YOUR-UNIQUE-ID/docker-compose.yml
wget https://setup.mailu.io/2024.06/file/YOUR-UNIQUE-ID/mailu.env
```

(Optional, for extra security): Add to the bottom of `/mailu/mailu.env`:

```
LD_PRELOAD=/usr/lib/libhardened_malloc.so
```

## Start Mailu

```bash
cd /mailu
docker compose -p mailu up -d
```

### Check running containers:

```bash
docker ps -a
docker compose logs -f front
```

### To restart or reset:

```bash
docker compose -p mailu down
docker compose -p mailu up -d
```

## DNS Records for Email

Add these records in Cloudflare (replace with your domain):

| Type  | Name           | Content                       | Extra      |
| ----- | -------------- | ----------------------------- | ---------- |
| MX    | @              | webmail.yourdomain.com        | Priority 10|
| TXT   | @              | v=spf1 mx a:webmail.yourdomain.com ~all | SPF        |
| TXT   | dkim.\_domainkey| (DKIM key from Mailu admin)   | DKIM       |
| TXT   | \_dmarc        | v=DMARC1; p=reject; rua=mailto:admin@yourdomain.com; ruf=mailto:admin@yourdomain.com; adkim=s; aspf=s | DMARC      |
| TLSA  | \_25.\_tcp.webmail| (TLSA data from Mailu admin)  | TLSA       |
| SRV   | \_autodiscover.\_tcp| webmail.yourdomain.com      | 10 1 443   |
| CNAME | autoconfig     | webmail.yourdomain.com        |            |
| CNAME | autodiscover     | webmail.yourdomain.com        |            |

Repeat for other SRV records as needed.

## First Login & Admin Setup

### Create admin user:

```bash
docker compose -p mailu exec admin flask mailu admin admin yourdomain.com YOUR_PASSWORD
```

Login at: `https://webmail.yourdomain.com`

Change the admin password immediately after first login.

## Backup System

Recommended: Use rsync or borgbackup for robust, incremental backups.

### A. Simple Rsync Backup Script

Create `/usr/local/bin/mailu-backup.sh`:

```bash
#!/bin/bash
# Mailu backup script

BACKUP_DIR="/var/backups/mailu"
MAILU_DATA="/mailu"
DATE=$(date +%F_%H-%M-%S)

mkdir -p "$BACKUP_DIR"
tar czf "$BACKUP_DIR/mailu_backup_$DATE.tar.gz" -C "$MAILU_DATA" .
find "$BACKUP_DIR" -type f -mtime +7 -delete  # Keep 7 days of backups
```

Make it executable:

```bash
sudo chmod +x /usr/local/bin/mailu-backup.sh
```

Add to cron (daily at 2am):

```bash
sudo crontab -e
# Add:
0 2 * * * /usr/local/bin/mailu-backup.sh
```

### B. Offsite Backup (Optional, Highly Recommended)

Add an rsync line to the script to sync to another server or cloud storage.

```bash
# After tar command in the script
rsync -avz --delete "$BACKUP_DIR/" user@backupserver:/remote/backup/mailu/
```

### C. Restore

To restore:

```bash
# Stop Mailu
cd /mailu
docker compose -p mailu down

# Restore backup
tar xzf /var/backups/mailu/mailu_backup_YYYY-MM-DD_HH-MM-SS.tar.gz -C /mailu

# Start Mailu
docker compose -p mailu up -d
```

## Troubleshooting

### Check logs:

```bash
docker compose logs -f
docker compose logs -f smtp
```

### Check if ports are in use:

```bash
sudo lsof -i :80
sudo lsof -i :443
```

### Remove all containers:

```bash
docker compose -p mailu down
```

### Recreate containers:

```bash
docker compose -p mailu up -d
