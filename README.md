# Complete iRedMail Installation & Configuration Guide for Ubuntu

## Prerequisites

- Ubuntu 20.04 LTS or 22.04 LTS (recommended)
- Minimum 2GB RAM (4GB+ recommended for production)
- Clean server installation (no existing mail services)
- Root or sudo access
- Valid domain name with DNS control
- Static IP address

## Important Notes Before Starting

⚠️ **Warning**: iRedMail will install and configure many components. Do NOT run this on a server with existing mail services or web applications.

✅ **Best Practice**: Use a dedicated server/VPS for mail services.

## Step 1: System Preparation

### 1.1 Update Your System
```bash
sudo apt update -y && sudo apt upgrade -y
sudo reboot
```

### 1.2 Set the Hostname
Replace `mail.example.com` with your actual mail subdomain:

```bash
sudo hostnamectl set-hostname mail.example.com
```

Verify the hostname (logout and login again to see the change):
```bash
hostname -f
```

### 1.3 Configure /etc/hosts
Edit the hosts file to ensure proper hostname resolution:

```bash
sudo nano /etc/hosts
```

Example configuration:
```
127.0.0.1   mail.example.com localhost
# Remove any other 127.0.0.1 entries
```

### 1.4 Install Required Utilities
```bash
sudo apt install -y gzip dialog wget curl
```

## Step 2: Download and Install iRedMail

### 2.1 Download Latest iRedMail
Check the [iRedMail releases page](https://github.com/iredmail/iRedMail/releases) for the latest version:

```bash
# Replace 1.6.8 with the latest version
wget https://github.com/iredmail/iRedMail/archive/refs/tags/1.6.8.tar.gz
tar xvf 1.6.8.tar.gz
cd iRedMail-1.6.8
chmod +x iRedMail.sh
```

### 2.2 Run the iRedMail Installer
```bash
sudo bash iRedMail.sh
```

**Installation Options:**
1. **Welcome Screen**: Press Enter to continue
2. **Mail Storage Path**: Accept default `/var/vmail`
3. **Web Server**: Choose **Nginx** (recommended)
4. **Database**: Choose **MariaDB** (recommended for most users)
5. **Database Password**: Set a strong password and save it securely
6. **Domain**: Enter your primary domain (e.g., `example.com`)
7. **Admin Password**: Set a strong password for postmaster@yourdomain.com
8. **Optional Components**: 
   - ✅ iRedAdmin (web-based admin panel)
   - ✅ Roundcube (webmail interface)
   - ✅ netdata (monitoring - optional)
   - ✅ Fail2ban (security)

### 2.3 Complete Installation
Review all settings and confirm. The installation will take 10-30 minutes depending on your server speed.

### 2.4 Reboot Server
```bash
sudo reboot
```

## Step 3: SSL/TLS Configuration with Let's Encrypt

### 3.1 Install Certbot
```bash
sudo apt update
sudo apt install -y snapd
sudo snap install core
sudo snap refresh core
sudo apt remove -y certbot  # Remove any existing certbot
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

### 3.2 Obtain SSL Certificate
Replace `mail.example.com` with your mail domain:

```bash
sudo certbot certonly --webroot --agree-tos --no-eff-email \
  -w /opt/www/well_known \
  -d mail.example.com
```

**Note**: You'll be prompted for an email address for certificate renewal notifications.

### 3.3 Configure Nginx for SSL
Edit the SSL template:

```bash
sudo nano /etc/nginx/templates/ssl.tmpl
```

Update the certificate paths:
```nginx
ssl_certificate /etc/letsencrypt/live/mail.example.com/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/mail.example.com/privkey.pem;
```

Test and reload Nginx:
```bash
sudo nginx -t
sudo systemctl reload nginx
```

### 3.4 Configure Postfix for SSL
Edit Postfix main configuration:

```bash
sudo nano /etc/postfix/main.cf
```

Update SSL certificate paths:
```
smtpd_tls_key_file = /etc/letsencrypt/live/mail.example.com/privkey.pem
smtpd_tls_cert_file = /etc/letsencrypt/live/mail.example.com/cert.pem
smtpd_tls_CAfile = /etc/letsencrypt/live/mail.example.com/chain.pem
smtpd_tls_CApath = /etc/letsencrypt/live/mail.example.com/
```

Reload Postfix:
```bash
sudo systemctl reload postfix
```

### 3.5 Configure Dovecot for SSL
Edit Dovecot configuration:

```bash
sudo nano /etc/dovecot/dovecot.conf
```

Update SSL certificate paths:
```
ssl_cert = </etc/letsencrypt/live/mail.example.com/fullchain.pem
ssl_key = </etc/letsencrypt/live/mail.example.com/privkey.pem
```

Reload Dovecot:
```bash
sudo systemctl reload dovecot
```

### 3.6 Setup Automatic Certificate Renewal
Create a renewal hook:

```bash
sudo nano /etc/letsencrypt/renewal-hooks/deploy/reload-mail-services.sh
```

Add the following content:
```bash
#!/bin/bash
systemctl reload nginx
systemctl reload postfix
systemctl reload dovecot
```

Make it executable:
```bash
sudo chmod +x /etc/letsencrypt/renewal-hooks/deploy/reload-mail-services.sh
```

Test automatic renewal:
```bash
sudo certbot renew --dry-run
```

## Step 4: DNS Configuration

### 4.1 Required DNS Records

**A Record:**
```
Type: A
Name: mail
Value: [Your server's IPv4 address]
TTL: 300
```

**MX Record:**
```
Type: MX
Name: @ (or your domain)
Value: mail.example.com
Priority: 10
TTL: 300
```

**PTR Record (Reverse DNS):**
- Configure through your hosting provider's control panel
- Point your server IP to `mail.example.com`
- This is crucial for email deliverability

### 4.2 Authentication Records

**SPF Record:**
```
Type: TXT
Name: @ (or your domain)
Value: v=spf1 mx ~all
TTL: 300
```

**DKIM Record:**
Generate the DKIM key:
```bash
sudo amavisd-new showkeys
```

Add the DKIM TXT record:
```
Type: TXT
Name: dkim._domainkey
Value: [Copy the public key from the command output, single line, no quotes]
TTL: 300
```

Test DKIM configuration:
```bash
sudo amavisd-new testkeys
```

If you get an error, restart Amavis:
```bash
sudo systemctl restart amavis
sudo amavisd-new testkeys
```

**DMARC Record:**
```
Type: TXT
Name: _dmarc
Value: v=DMARC1; p=quarantine; pct=100; rua=mailto:dmarc-reports@example.com; ruf=mailto:dmarc-failures@example.com; fo=1
TTL: 300
```

### 4.3 Verify DNS Propagation
Use online tools to verify your DNS records:
- [MXToolbox](https://mxtoolbox.com/)
- [DNS Checker](https://dnschecker.org/)

**Note**: DNS propagation can take 4-48 hours.

## Step 5: Initial Configuration and Testing

### 5.1 Access Admin Panel
Navigate to: `https://mail.example.com/iredadmin/`

Login with:
- Username: `postmaster@example.com`
- Password: [Admin password set during installation]

### 5.2 Create Mail Users
1. Click **"Add"** → **"User"**
2. Fill in the form:
   - Email: `user@example.com`
   - Password: [Strong password]
   - Display Name: `User Name`
   - Quota: `1024` (MB) or as needed

### 5.3 Test Webmail
1. Go to: `https://mail.example.com/mail/`
2. Login with the user credentials
3. Send a test email to verify functionality

### 5.4 Test Email Deliverability
Send test emails to:
- [Mail Tester](https://www.mail-tester.com/) - Check spam score
- [Gmail](https://gmail.com/) - Test major provider delivery
- [Outlook](https://outlook.com/) - Test Microsoft delivery

## Step 6: Security Hardening

### 6.1 Firewall Configuration
```bash
# Install UFW if not installed
sudo apt install -y ufw

# Allow SSH (adjust port if needed)
sudo ufw allow 22/tcp

# Allow mail services
sudo ufw allow 25/tcp    # SMTP
sudo ufw allow 587/tcp   # SMTP Submission
sudo ufw allow 993/tcp   # IMAPS
sudo ufw allow 995/tcp   # POP3S
sudo ufw allow 80/tcp    # HTTP (for Let's Encrypt)
sudo ufw allow 443/tcp   # HTTPS

# Enable firewall
sudo ufw --force enable
```

### 6.2 Fail2Ban Configuration
Check Fail2Ban status:
```bash
sudo systemctl status fail2ban
sudo fail2ban-client status
```

### 6.3 Regular Updates
Create an update script:
```bash
sudo nano /root/update-system.sh
```

Add:
```bash
#!/bin/bash
apt update
apt upgrade -y
apt autoremove -y
systemctl restart postfix dovecot nginx
```

Make executable and add to cron:
```bash
sudo chmod +x /root/update-system.sh
sudo crontab -e
```

Add weekly updates (Sundays at 2 AM):
```
0 2 * * 0 /root/update-system.sh
```

## Step 7: Monitoring and Maintenance

### 7.1 Log Locations
- **Postfix**: `/var/log/mail.log`
- **Dovecot**: `/var/log/mail.log`
- **Nginx**: `/var/log/nginx/`
- **iRedMail**: `/var/log/iredmail/`

### 7.2 Useful Commands
```bash
# Check service status
sudo systemctl status postfix dovecot nginx amavis clamav-daemon

# View mail queue
sudo postqueue -p

# Clear mail queue
sudo postsuper -d ALL

# Check disk usage
df -h
du -sh /var/vmail/*

# Monitor real-time logs
sudo tail -f /var/log/mail.log
```

### 7.3 Backup Strategy
Create a backup script:
```bash
sudo nano /root/backup-mail.sh
```

Add:
```bash
#!/bin/bash
BACKUP_DIR="/backup/mail/$(date +%Y-%m-%d)"
mkdir -p $BACKUP_DIR

# Backup mail data
tar -czf $BACKUP_DIR/vmail.tar.gz /var/vmail/

# Backup databases
mysqldump --all-databases | gzip > $BACKUP_DIR/databases.sql.gz

# Backup configuration
tar -czf $BACKUP_DIR/configs.tar.gz /etc/postfix/ /etc/dovecot/ /etc/nginx/ /etc/amavis/

# Remove backups older than 7 days
find /backup/mail/ -type d -mtime +7 -exec rm -rf {} +
```

## Step 8: Troubleshooting

### 8.1 Common Issues

**Issue**: Emails going to spam
- **Solution**: Verify SPF, DKIM, and DMARC records
- Check reverse DNS (PTR record)
- Use mail tester tools

**Issue**: Cannot receive emails
- **Solution**: Check MX records
- Verify firewall allows port 25
- Check `/var/log/mail.log` for errors

**Issue**: SSL certificate errors
- **Solution**: Verify certificate paths in configs
- Check certificate expiration: `sudo certbot certificates`

### 8.2 Performance Optimization

For high-volume mail servers:
```bash
# Edit Postfix main.cf
sudo nano /etc/postfix/main.cf
```

Add/modify:
```
# Performance tuning
default_process_limit = 100
smtpd_client_connection_count_limit = 50
smtpd_client_connection_rate_limit = 30
```

## Step 9: Additional Features

### 9.1 Enable Postfix Admin (Alternative Web Interface)
```bash
# Download and install Postfix Admin if needed
wget https://github.com/postfixadmin/postfixadmin/archive/postfixadmin-3.3.tar.gz
```

### 9.2 Mobile Configuration
Provide users with these settings:

**IMAP Settings:**
- Server: mail.example.com
- Port: 993
- Security: SSL/TLS

**SMTP Settings:**
- Server: mail.example.com
- Port: 587
- Security: STARTTLS
- Authentication: Required

## Step 10: Advanced Configuration

### 10.1 Multiple Domains
To add additional domains:
1. Access iRedAdmin
2. Go to **"Add"** → **"Domain"**
3. Configure DNS records for the new domain
4. Add users for the new domain

### 10.2 Aliases and Forwarding
Create aliases in iRedAdmin:
1. **"Add"** → **"Alias"**
2. Set source and destination addresses

## Maintenance Checklist

### Daily
- [ ] Check server resources (CPU, RAM, Disk)
- [ ] Monitor mail queue size
- [ ] Check for failed login attempts

### Weekly
- [ ] Review mail logs for errors
- [ ] Check SSL certificate status
- [ ] Verify backup completion
- [ ] Test email deliverability

### Monthly
- [ ] Update system packages
- [ ] Review user accounts and quotas
- [ ] Check DNS record status
- [ ] Analyze mail server performance

## Support Resources

- **iRedMail Documentation**: [https://docs.iredmail.org/](https://docs.iredmail.org/)
- **iRedMail Forum**: [https://forum.iredmail.org/](https://forum.iredmail.org/)
- **Ubuntu Documentation**: [https://help.ubuntu.com/](https://help.ubuntu.com/)

## Conclusion

You now have a fully functional mail server with iRedMail. Remember to:

1. Keep your system updated
2. Monitor logs regularly
3. Maintain proper backups
4. Test email deliverability periodically
5. Keep SSL certificates current

For production use, consider implementing additional monitoring tools and establishing proper backup procedures.

---

**Created by**: [Your Name]  
**Last Updated**: [Current Date]  
**Version**: 2.0

---

*This guide is based on iRedMail 1.6.8 and Ubuntu 22.04 LTS. Always refer to the official iRedMail documentation for the latest updates and specific configuration details.*