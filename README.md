# 📧 Professional Email System Setup Guide

Complete guides for setting up production-ready email servers on Ubuntu with SSL, DNS configuration, and security hardening.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-20.04%20|%2022.04-orange.svg)](https://ubuntu.com/)
[![Docker](https://img.shields.io/badge/Docker-Supported-blue.svg)](https://www.docker.com/)

## 🚀 Quick Start

```bash
# Clone the repository
git clone https://github.com/kadavilrahul/setup_email_system.git

# Navigate to the directory
cd setup_email_system
```

## 📋 Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Setup Options](#setup-options)
- [Quick Installation](#quick-installation)
- [Detailed Guides](#detailed-guides)
- [DNS Configuration](#dns-configuration)
- [Security & Maintenance](#security--maintenance)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [Support](#support)

## 🔍 Overview

This repository provides two comprehensive email server setup solutions:

### 🏛️ iRedMail (Traditional LAMP Stack)
- **Best for**: Traditional server environments, full control
- **Technology**: LAMP stack (Linux, Apache/Nginx, MySQL, PHP)
- **Complexity**: Medium to Advanced
- **Resources**: 2GB+ RAM recommended
- **Management**: Web-based admin panel + command line

### 🐳 Mailu (Modern Docker-based)
- **Best for**: Modern containerized environments, easy maintenance
- **Technology**: Docker containers
- **Complexity**: Beginner to Medium
- **Resources**: 1GB+ RAM sufficient
- **Management**: Web-based admin panel

## 📦 Prerequisites

### System Requirements

| Component | iRedMail | Mailu |
|-----------|----------|-------|
| **OS** | Ubuntu 20.04/22.04 LTS | Ubuntu 20.04/22.04 LTS |
| **RAM** | 2GB+ (4GB+ recommended) | 1GB+ (2GB+ recommended) |
| **Storage** | 20GB+ | 10GB+ |
| **Network** | Static IP, Port 25 unblocked | Static IP, Port 25 unblocked |

### Required Access
- ✅ Root or sudo access to server
- ✅ Domain name with DNS control
- ✅ Clean Ubuntu installation (no existing mail services)
- ✅ Static IP address with reverse DNS (PTR) capability

### Supported Hosting Providers
- **Hetzner Cloud** ⭐ (Recommended)
- **Contabo VPS** ⭐ (Recommended)
- **DigitalOcean**
- **Vultr**
- **Linode**
- **AWS EC2** (with proper configuration)

## 🛠️ Setup Options

### Option 1: iRedMail Setup (Traditional)

**Features:**
- Complete LAMP stack installation
- Postfix + Dovecot configuration
- Automatic SSL with Let's Encrypt
- Spam filtering with SpamAssassin
- Antivirus with ClamAV
- Web-based administration
- Roundcube webmail

### Option 2: Mailu Setup (Docker-based)

**Features:**
- Containerized email stack
- Easy backup and migration
- Built-in Let's Encrypt SSL
- Modern web interface
- Automatic updates
- Resource efficient

## 🚀 Quick Installation

### Automated Installation (Recommended for beginners)

1. **Clone the repository:**
   ```bash
   git clone https://github.com/kadavilrahul/setup_email_system.git
   cd setup_email_system
   ```

2. **Run the interactive installer:**

3. **Follow the prompts:**
   - Choose your preferred email system (iRedMail or Mailu)
   - Enter your domain name
   - Set admin credentials
   - Configure SSL settings


## 📚 Detailed Guides

### 📖 Available Documentation

- **[iRedMail Complete Guide](guides/iredmail-complete-guide.md)** - Comprehensive installation and configuration
- **[Mailu Setup Guide](guides/mailu-setup-guide.md)** - Docker-based email server setup
- **[DNS Configuration](guides/dns-configuration.md)** - Complete DNS setup for email servers
- **[SSL Configuration](guides/ssl-setup.md)** - Let's Encrypt SSL certificate setup
- **[Security Hardening](guides/security-hardening.md)** - Server security best practices
- **[Backup & Recovery](guides/backup-recovery.md)** - Data protection strategies
- **[Troubleshooting](guides/troubleshooting.md)** - Common issues and solutions

### 🎯 Quick Reference

| Task | iRedMail | Mailu |
|------|----------|-------|
| **Admin Panel** | `https://mail.domain.com/iredadmin/` | `https://webmail.domain.com/admin/` |
| **Webmail** | `https://mail.domain.com/mail/` | `https://webmail.domain.com/` |
| **Logs** | `/var/log/mail.log` | `docker compose logs` |
| **Restart Services** | Multiple systemctl commands | `docker compose restart` |
| **Backup** | File system + MySQL | Docker volumes |

## 🌐 DNS Configuration

### Required DNS Records

```dns
# A Records
mail.yourdomain.com.     IN  A       YOUR_SERVER_IP
webmail.yourdomain.com.  IN  A       YOUR_SERVER_IP

# MX Record
yourdomain.com.          IN  MX  10  mail.yourdomain.com.

# SPF Record
yourdomain.com.          IN  TXT     "v=spf1 mx ~all"

# DKIM Record (generated after setup)
dkim._domainkey.yourdomain.com. IN TXT "v=DKIM1; k=rsa; p=YOUR_PUBLIC_KEY"

# DMARC Record
_dmarc.yourdomain.com.   IN  TXT     "v=DMARC1; p=quarantine; rua=mailto:dmarc@yourdomain.com"
```

### DNS Providers Configuration

- **[Cloudflare Setup](dns-guides/cloudflare.md)**
- **[Namecheap Setup](dns-guides/namecheap.md)**
- **[GoDaddy Setup](dns-guides/godaddy.md)**
- **[Route53 Setup](dns-guides/route53.md)**

## 🔒 Security & Maintenance

### Security Features Included

- ✅ **SSL/TLS Encryption** - Let's Encrypt certificates
- ✅ **Firewall Configuration** - UFW with proper mail ports
- ✅ **Fail2Ban Protection** - Brute force protection
- ✅ **Spam Filtering** - SpamAssassin/Rspamd
- ✅ **Antivirus Scanning** - ClamAV integration
- ✅ **DKIM/SPF/DMARC** - Email authentication


## 🔧 Configuration Files

### Important Configuration Locations

#### iRedMail
```
/etc/postfix/main.cf      # Postfix main configuration
/etc/dovecot/dovecot.conf # Dovecot configuration
/etc/nginx/              # Web server configuration
/opt/iredmail/           # iRedMail specific configs
```

#### Mailu
```
/mailu/docker-compose.yml # Docker composition
/mailu/mailu.env         # Environment variables
/mailu/data/             # Mail data directory
/mailu/certs/            # SSL certificates
```

## 📊 Monitoring & Analytics

### Built-in Monitoring

- **Server Resources** - CPU, RAM, Disk usage
- **Email Queue** - Pending/failed emails
- **Security Events** - Failed login attempts
- **SSL Certificate** - Expiration monitoring

### Optional Monitoring Tools

```bash
# Install monitoring stack
./scripts/install-monitoring.sh

# Includes:
# - Prometheus for metrics
# - Grafana for dashboards  
# - AlertManager for notifications
```

## 🆘 Troubleshooting

### Common Issues

1. **Emails going to spam**
   - Verify SPF, DKIM, DMARC records
   - Check reverse DNS (PTR record)
   - Use [Mail Tester](https://www.mail-tester.com/)

2. **Cannot receive emails**
   - Check MX records
   - Verify firewall allows port 25
   - Check mail server logs

3. **SSL certificate issues**
   - Verify domain DNS points to server
   - Check Let's Encrypt rate limits
   - Review certificate configuration

### Debug Commands

```bash
# Check email server status
./scripts/check-status.sh

# Test email delivery
./scripts/test-email.sh user@domain.com

# View real-time logs
./scripts/tail-logs.sh

# Verify DNS records
./scripts/check-dns.sh yourdomain.com
```

## 🏢 Production Deployment

### Pre-deployment Checklist

- [ ] Server meets minimum requirements
- [ ] Domain DNS configured correctly
- [ ] Reverse DNS (PTR) record set
- [ ] Port 25 unblocked by hosting provider
- [ ] SSL certificate domain validated
- [ ] Backup strategy implemented
- [ ] Monitoring setup completed

### Performance Tuning

```bash
# Optimize for high-volume email
./scripts/performance-tune.sh

# Configure for specific use cases:
./scripts/tune-for-marketing.sh    # Marketing emails
./scripts/tune-for-transactional.sh # Transactional emails
./scripts/tune-for-general.sh     # General business use
```

## 📱 Client Configuration

### Email Client Settings

#### IMAP Settings
```
Server: mail.yourdomain.com
Port: 993
Encryption: SSL/TLS
Authentication: Normal password
```

#### SMTP Settings
```
Server: mail.yourdomain.com
Port: 587
Encryption: STARTTLS
Authentication: Normal password
```

### Mobile Setup Guides
- **[iOS Mail Setup](client-guides/ios-mail.md)**
- **[Android Gmail Setup](client-guides/android-gmail.md)**
- **[Outlook Setup](client-guides/outlook.md)**
- **[Thunderbird Setup](client-guides/thunderbird.md)**

## 🔄 Migration & Backup

### Email Migration Tools

```bash
# Migrate from existing email server
./scripts/migrate-emails.sh

# Supported sources:
# - cPanel/WHM servers
# - Plesk servers  
# - Exchange servers
# - Gmail/Google Workspace
# - Other IMAP servers
```


## 🤝 Contributing

We welcome contributions! Here's how you can help:

1. **Fork the repository**
2. **Create a feature branch**
   ```bash
   git checkout -b feature/new-email-provider
   ```
3. **Make your changes**
4. **Test thoroughly**
5. **Submit a pull request**


## 📞 Support

### Community Support
- **GitHub Issues**: [Report bugs or request features](https://github.com/kadavilrahul/setup_email_system/issues)
- **Discussions**: [Community discussions](https://github.com/kadavilrahul/setup_email_system/discussions)

### Professional Support
For business implementations and custom configurations:
- **Email**: support@yourcompany.com
- **Website**: [Your AI Agent Services](https://yourwebsite.com)

### Documentation
- **Wiki**: [Detailed documentation](https://github.com/kadavilrahul/setup_email_system/wiki)
- **FAQ**: [Frequently Asked Questions](FAQ.md)

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🌟 Star History

If this repository helped you set up your email server, please consider giving it a star! ⭐

## 🔗 Related Projects

- **[E-commerce Tools](https://github.com/kadavilrahul/ecommerce-tools)** - WooCommerce optimization tools
- **[Server Management Scripts](https://github.com/kadavilrahul/server-scripts)** - Ubuntu server management utilities
- **[AI Agent Services](https://github.com/kadavilrahul/ai-agent-services)** - Business automation solutions

## 📈 Roadmap

### Upcoming Features
- [ ] One-click DigitalOcean deployment
- [ ] Kubernetes deployment manifests
- [ ] Advanced spam filtering with AI
- [ ] Multi-domain management interface
- [ ] Email marketing integration
- [ ] Advanced monitoring dashboard

### Version History
- **v2.0.0** - Added Mailu support, improved security
- **v1.5.0** - Enhanced DNS configuration, added monitoring
- **v1.0.0** - Initial iRedMail implementation

---

**Made with ❤️ for the email server community**

*If you found this repository helpful, please consider supporting the project by starring it and sharing with others who might need it.*
