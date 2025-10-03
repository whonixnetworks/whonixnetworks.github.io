---
title: Linux System Hardening Guide
description: >
  Learn essential Linux system hardening techniques to secure your servers. This guide covers SSH security, firewall configuration, user management, and more.
author: greedy
date: 2025-10-03 16:00:00 +1030
categories: [Homelab, Linux, Security]
tags: [linux, security, hardening, ssh, firewall]
pin: false
---

Securing your Linux systems is crucial in today's interconnected world. Whether you're running a homelab, web server, or production infrastructure, proper hardening protects against unauthorized access and potential attacks. This guide covers essential Linux system hardening techniques that every administrator should implement.

## What You'll Learn

By following this guide, you'll implement:

- SSH security enhancements and key-based authentication
- Firewall configuration with UFW or iptables
- User account security and privilege management
- System service hardening and unnecessary service removal
- Security auditing and monitoring
- Automated security updates

## Prerequisites

Before starting, ensure you have:

- A Linux server (Ubuntu/CentOS/Debian)
- Root or sudo access
- Basic familiarity with Linux command line
- Backup of critical data and configuration

> **Warning:** Some of these changes can lock you out of your system if misconfigured. Always test in a non-production environment first and ensure you have console access as a backup.
{: .prompt-warning }

---

## Initial Security Assessment

Start by assessing your current system security:

```bash
# Check currently logged in users
who
w

# Check listening ports
ss -tuln
netstat -tuln

# Check running processes
ps aux

# Check failed login attempts
sudo lastb | head -20

# Check system information
uname -a
cat /etc/os-release
```

SSH Hardening

Use Key-Based Authentication

Disable password authentication and use SSH keys:

```bash
# Generate SSH key pair (on your local machine)
ssh-keygen -t ed25519 -f ~/.ssh/server_key

# Copy public key to server
ssh-copy-id -i ~/.ssh/server_key.pub user@your-server

# Test key-based login
ssh -i ~/.ssh/server_key user@your-server
```

Secure SSH Configuration

Edit /etc/ssh/sshd_config:

```bash
sudo nano /etc/ssh/sshd_config
```

Apply these settings:

```bash
# Disable root login
PermitRootLogin no

# Use only SSH protocol 2
Protocol 2

# Disable password authentication
PasswordAuthentication no

# Disable empty passwords
PermitEmptyPasswords no

# Maximum authentication attempts
MaxAuthTries 3

# Login grace time
LoginGraceTime 60

# Allow specific users only (optional)
AllowUsers your_username admin_user

# Disable X11 forwarding
X11Forwarding no

# Use stronger key exchange algorithms
KexAlgorithms curve25519-sha256@libssh.org
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com

# Change default port (optional)
Port 2222
```

When changing SSH port, ensure your firewall allows the new port and test connectivity before disconnecting.
{:.prompt-tip }

Restart SSH service:

```bash
sudo systemctl restart sshd
# Test from another session before closing current connection
```

Firewall Configuration

UFW (Ubuntu/Debian)

```bash
# Enable UFW
sudo ufw enable

# Allow SSH (adjust port if changed)
sudo ufw allow 22/tcp

# Allow HTTP/HTTPS if running web services
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Default deny incoming, allow outgoing
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Check status
sudo ufw status verbose
```

firewalld (CentOS/RHEL)

```bash
# Start and enable firewalld
sudo systemctl enable firewalld
sudo systemctl start firewalld

# Allow services
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https

# Reload firewall
sudo firewall-cmd --reload

# Check status
sudo firewall-cmd --list-all
```

iptables (Advanced)

Create a basic iptables ruleset:

```bash
#!/bin/bash
# Clear existing rules
iptables -F
iptables -X

# Set default policies
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Allow loopback
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH (adjust port as needed)
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP/HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow ICMP (ping)
iptables -A INPUT -p icmp -j ACCEPT

# Save rules (distribution specific)
iptables-save > /etc/iptables/rules.v4
```

User Account Security

Create Limited Privilege Users

```bash
# Create new user
sudo adduser newusername

# Add to sudo group (if needed)
sudo usermod -aG sudo newusername

# Set strong password
sudo passwd newusername
```

Configure Password Policies

Edit /etc/security/pwquality.conf or /etc/pam.d/common-password:

```bash
# Minimum password length
minlen = 12

# Require multiple character classes
minclass = 3

# Password history
remember = 5
```

Configure password aging in /etc/login.defs:

```bash
PASS_MAX_DAYS 90
PASS_MIN_DAYS 7
PASS_WARN_AGE 14
```

Apply to existing users:

```bash
sudo chage -M 90 -m 7 -W 14 username
```

Configure Sudo Security

Edit /etc/sudoers with visudo:

```bash
# Require password for sudo
Defaults passwd_timeout=5

# Log all sudo commands
Defaults logfile="/var/log/sudo.log"

# Limit sudo to specific commands for specific users
username ALL=(ALL) /bin/systemctl, /usr/bin/apt
```

System Service Hardening

Disable Unnecessary Services

```bash
# List running services
sudo systemctl list-units --type=service --state=running

# Disable and stop unnecessary services
sudo systemctl stop bluetooth
sudo systemctl disable bluetooth

sudo systemctl stop cups
sudo systemctl disable cups

# Common services to consider disabling if not needed:
# - bluetooth
# - cups (printing)
# - avahi-daemon (mDNS)
# - rpcbind (NFS)
```

Secure Shared Memory

Edit /etc/fstab:

```bash
# Add this line to secure shared memory
tmpfs /dev/shm tmpfs defaults,noexec,nosuid 0 0
```

Remount:

```bash
sudo mount -o remount /dev/shm
```

Kernel Hardening

Configure sysctl Parameters

Edit /etc/sysctl.conf or create /etc/sysctl.d/99-security.conf:

```bash
# Network security
net.ipv4.ip_forward=0
net.ipv4.conf.all.send_redirects=0
net.ipv4.conf.default.send_redirects=0
net.ipv4.conf.all.accept_redirects=0
net.ipv4.conf.default.accept_redirects=0
net.ipv4.conf.all.accept_source_route=0
net.ipv4.conf.default.accept_source_route=0
net.ipv4.conf.all.log_martians=1
net.ipv4.conf.default.log_martians=1
net.ipv4.icmp_echo_ignore_broadcasts=1
net.ipv4.icmp_ignore_bogus_error_responses=1
net.ipv4.tcp_syncookies=1
net.ipv6.conf.all.accept_redirects=0
net.ipv6.conf.default.accept_redirects=0

# Memory protection
kernel.exec-shield=1
kernel.randomize_va_space=2

# Restrict core dumps
fs.suid_dumpable=0

# Kernel hardening
kernel.kptr_restrict=2
kernel.dmesg_restrict=1
kernel.yama.ptrace_scope=1
```

Apply changes:

```bash
sudo sysctl -p
```

File System Security

Configure File Permissions

```bash
# Set strict permissions on sensitive files
sudo chmod 600 /etc/shadow
sudo chmod 600 /etc/gshadow
sudo chmod 644 /etc/passwd
sudo chmod 644 /etc/group

# Set sticky bit on world-writable directories
sudo chmod +t /tmp
sudo chmod +t /var/tmp

# Secure cron directories
sudo chmod -R go-rwx /etc/cron*
```

Install and Configure Fail2ban

Protect against brute force attacks:

```bash
# Install fail2ban
sudo apt install fail2ban  # Ubuntu/Debian
sudo yum install fail2ban  # CentOS/RHEL

# Create local configuration
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

Edit /etc/fail2ban/jail.local:

```bash
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 3

[sshd]
enabled = true
port = ssh
logpath = /var/log/auth.log
maxretry = 3

[sshd-ddos]
enabled = true
port = ssh
logpath = /var/log/auth.log
maxretry = 5
```

Start and enable fail2ban:

```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# Check status
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

Automatic Security Updates

Ubuntu/Debian

```bash
# Install unattended-upgrades
sudo apt install unattended-upgrades

# Configure automatic security updates
sudo dpkg-reconfigure -plow unattended-upgrades

# Or edit /etc/apt/apt.conf.d/50unattended-upgrades
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

Ensure these lines are present:

```bash
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
};
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-Time "02:00";
```

CentOS/RHEL

```bash
# Install dnf-automatic
sudo dnf install dnf-automatic

# Enable security updates only
sudo nano /etc/dnf/automatic.conf
```

Set:

```bash
upgrade_type = security
apply_updates = yes
random_sleep = 300
```

Enable and start:

```bash
sudo systemctl enable --now dnf-automatic.timer
```

Security Auditing and Monitoring

Install and Configure Auditd

```bash
# Install auditd
sudo apt install auditd audispd-plugins  # Ubuntu/Debian
sudo yum install audit audispd-plugins   # CentOS/RHEL

# Start and enable
sudo systemctl enable auditd
sudo systemctl start auditd
```

Configure audit rules in /etc/audit/audit.rules:

```bash
# Monitor file changes in critical directories
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/gshadow -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/sudoers -p wa -k identity

# Monitor system calls
-a always,exit -F arch=b64 -S chmod -S fchmod -S fchmodat -F auid>=1000 -F auid!=4294967295 -k perm_mod
-a always,exit -F arch=b64 -S chown -S fchown -S fchownat -S lchown -F auid>=1000 -F auid!=4294967295 -k perm_mod

# Monitor network configuration
-w /etc/network/ -p wa -k network_mod
```

Install and Run Lynis

```bash
# Download and install Lynis
wget https://downloads.cisofy.com/lynis/lynis-3.0.8.tar.gz
tar xvf lynis-3.0.8.tar.gz
cd lynis-3.0.8/

# Run system audit
sudo ./lynis audit system

# View report
sudo cat /var/log/lynis-report.dat
```

Application Security

Harden Web Servers

For Apache:

```bash
# Edit /etc/apache2/conf-available/security.conf
sudo nano /etc/apache2/conf-available/security.conf
```

Set:

```bash
ServerTokens Prod
ServerSignature Off
TraceEnable Off
FileETag None
```

For Nginx:

```bash
# Edit /etc/nginx/nginx.conf
sudo nano /etc/nginx/nginx.conf
```

Add to http section:

```bash
server_tokens off;
add_header X-Frame-Options SAMEORIGIN;
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";
```

Database Security

For MySQL/MariaDB:

```bash
# Run security script
sudo mysql_secure_installation

# Create application-specific users
mysql -u root -p
```

```sql
CREATE USER 'appuser'@'localhost' IDENTIFIED BY 'strong_password';
GRANT SELECT, INSERT, UPDATE, DELETE ON database.* TO 'appuser'@'localhost';
FLUSH PRIVILEGES;
```

Backup and Recovery

System Configuration Backups

```bash
# Create backup of critical configuration files
sudo tar -czf /backup/system-config-$(date +%Y%m%d).tar.gz \
  /etc/ssh/sshd_config \
  /etc/ufw/ \
  /etc/fail2ban/ \
  /etc/audit/ \
  /etc/sysctl.conf \
  /etc/sysctl.d/
```

Automated Backups Script

```bash
#!/bin/bash
# backup-security-config.sh

BACKUP_DIR="/backup/security-config"
DATE=$(date +%Y%m%d)

mkdir -p $BACKUP_DIR

# Backup critical security files
tar -czf $BACKUP_DIR/security-config-$DATE.tar.gz \
  /etc/ssh/ \
  /etc/ufw/ \
  /etc/fail2ban/ \
  /etc/audit/ \
  /etc/pam.d/ \
  /etc/sudoers.d/ \
  /etc/sysctl.d/

# Set permissions
chmod 600 $BACKUP_DIR/security-config-$DATE.tar.gz

# Remove backups older than 30 days
find $BACKUP_DIR -name "security-config-*.tar.gz" -mtime +30 -delete

echo "Security configuration backup completed: $BACKUP_DIR/security-config-$DATE.tar.gz"
```

Monitoring and Alerting

Set Up Log Monitoring

```bash
# Install logwatch
sudo apt install logwatch  # Ubuntu/Debian
sudo yum install logwatch  # CentOS/RHEL

# Configure daily reports
sudo nano /etc/logwatch/conf/logwatch.conf
```

Set:

```bash
Output = mail
Format = html
MailTo = your-email@example.com
Range = yesterday
Detail = High
```

Security Script for Daily Checks

```bash
#!/bin/bash
# security-daily-check.sh

echo "=== Daily Security Check ==="
echo "Date: $(date)"
echo

echo "1. Failed login attempts:"
sudo lastb | head -10
echo

echo "2. Currently logged in users:"
who
echo

echo "3. Listening ports:"
ss -tuln | grep LISTEN
echo

echo "4. Failed SSH attempts:"
sudo grep "Failed password" /var/log/auth.log | wc -l
echo

echo "5. UFW status:"
sudo ufw status verbose
echo

echo "6. Fail2ban status:"
sudo fail2ban-client status
echo

echo "7. System updates available:"
apt list --upgradable 2>/dev/null | wc -l
```

Troubleshooting

Common Issues and Solutions

Locked out of SSH: Use console access to verify SSH configuration and firewall rules. Check if you're using the correct port and key.
{:.prompt-warning }

Services not starting: Check system logs with journalctl -u service-name and verify configuration files for syntax errors.
{:.prompt-warning }

Firewall blocking legitimate traffic: Temporarily disable firewall for testing: sudo ufw disable or sudo systemctl stop firewalld.
{:.prompt-warning }

Recovery Steps

If you get locked out:

1. Use console/out-of-band access
2. Boot into single-user/rescue mode if needed
3. Check and fix configuration files
4. Restart services one by one
5. Test connectivity before closing recovery session

Maintenance Schedule

Daily

· Check security alerts and logs
· Monitor failed login attempts
· Review system resource usage

Weekly

· Apply security updates
· Review audit logs
· Verify backup integrity
· Run security scans

Monthly

· Review user accounts and permissions
· Update firewall rules if needed
· Perform comprehensive system audit
· Test recovery procedures

Next Steps

· Implement centralized logging with rsyslog
· Set up intrusion detection with AIDE
· Configure SELinux or AppArmor
· Implement two-factor authentication
· Set up security monitoring dashboard
· Regular penetration testing
· Security certification compliance (CIS benchmarks)

Remember: Security is an ongoing process, not a one-time setup. Regular maintenance, monitoring, and updates are essential for maintaining a secure system.