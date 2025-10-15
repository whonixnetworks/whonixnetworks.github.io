---
title: Setting Up A Linux Server
description: >
  A step-by-step guide for beginners to set up a Linux server on Debian/Ubuntu. Learn essential setup steps, security best practices, and how to avoid common pitfalls.
author: greedy
date: 2025-10-10
categories: [Linux-Guides, Getting-Started]
tags: [linux, server, beginner, setup, security, ubuntu, debian]
pin: true
---

> **INFO:** This guide is designed for Debian or Ubuntu-based Linux distributions. While many concepts apply to other distros, specific commands and tools may vary. Always verify compatibility with your chosen distribution.
{:.prompt-info }

![Desktop View](/assets/img/linuxsetup.png){: .shadow style="border-radius:16px" }

Setting up your first Linux server can feel like diving into the deep end—especially if you're new to the command line or server management. The good news? With a clear plan and some basic steps, you can have a secure, functional server up and running without tearing your hair out. This guide is for beginners, so I'll walk you through the essentials, from initial setup to securing your server, while keeping things practical and human-friendly.

Let’s get your server ready to roll.

---

## What You'll Need to Start

- **Hardware**: Any x86_64 computer with at least 4GB RAM and a 20GB+ SSD (e.g., an old laptop, mini PC, or even a Raspberry Pi).
- **OS**: A fresh install of Debian or Ubuntu Server (LTS versions like Ubuntu 24.04 or Debian 12 are great for stability).
- **Network**: A stable internet connection and access to your router for port forwarding (if needed).
- **Time**: About 1-2 hours for the initial setup, plus time to tinker and learn.

> **TIP**: If you're using a virtual machine to test, tools like VirtualBox or Proxmox are great. Start with 2 CPU cores, 4GB RAM, and a 20GB disk.
{:.prompt-tip }

---

## Step 1: Initial Setup and Updates

After installing Debian or Ubuntu Server, your first task is to ensure the system is up to date. This patches security vulnerabilities and ensures you have the latest software.

### Update and Upgrade Packages

Log in as the root user (or the user created during installation) and run:

```bash
# Update the package lists
sudo apt update

# Upgrade installed packages
sudo apt upgrade -y

# Optionally, install all available updates (including kernel)
sudo apt full-upgrade -y
```

> **TIP**: The `-y` flag automatically confirms prompts, but you can omit it to review changes manually.
{:.prompt-tip }

### Install Essential Packages

You’ll want a few basic tools to make life easier:

```bash
# Install commonly used utilities
sudo apt install -y curl wget vim nano git net-tools htop
```

- `curl` and `wget`: For downloading files or interacting with APIs.
- `vim` or `nano`: Text editors for configuration files.
- `git`: For version control and pulling code.
- `net-tools` and `htop`: For network diagnostics and system monitoring.

---

## Step 2: Setting Up Users and Permissions

Running everything as the root user is a security risk. Instead, create a dedicated user with sudo privileges for daily tasks.

### Create a New User

```bash
# Add a new user (replace 'username' with your preferred name)
sudo adduser username

# Add the user to the sudo group
sudo usermod -aG sudo username
```

### Verify Sudo Access

Switch to the new user and test sudo:

```bash
# Switch to the new user
su - username

# Test sudo privileges
sudo apt update
```

> **WARNING**: Never share your user’s password, and avoid using root for routine tasks. Sudo keeps things safer.
{:.prompt-danger }

---

## Step 3: Securing SSH Access

SSH (Secure Shell) lets you access your server remotely. By default, it’s enabled on most Linux servers, but the default settings aren’t always secure.

### SSH Best Practices

1. **Change the Default SSH Port** (optional but recommended):
   Edit the SSH configuration file to use a non-standard port (e.g., 2222) to reduce automated attacks.

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

   Find the line `#Port 22` and change it to:

   ```bash
   Port 2222
   ```

2. **Disable Root Login**:
   In the same file (`/etc/ssh/sshd_config`), find and set:

   ```bash
   PermitRootLogin no
   ```

3. **Enable Public Key Authentication**:
   Passwords can be brute-forced, so use SSH keys for better security.

   **On your local machine** (not the server):
   Generate an SSH key pair if you don’t have one:

   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   ```

   Copy the public key to your server:

   ```bash
   ssh-copy-id -i ~/.ssh/id_ed25519.pub username@your_server_ip -p 2222
   ```

   **On the server**:
   Disable password authentication in `/etc/ssh/sshd_config`:

   ```bash
   PasswordAuthentication no
   ```

4. **Restart SSH**:
   Apply the changes:

   ```bash
   sudo systemctl restart sshd
   ```

> **TIP**: Always test your SSH configuration from another terminal before closing your current session to avoid being locked out.
{:.prompt-tip }

---

## Step 4: Configuring the Firewall with UFW

Uncomplicated Firewall (UFW) is a beginner-friendly way to secure your server by controlling network traffic.

### Install and Enable UFW

```bash
sudo apt install -y ufw
```

### Set Default Rules

Deny all incoming connections by default, but allow outgoing:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### Allow Essential Services

Allow SSH (on your custom port, if changed) and other services you plan to run:

```bash
# Allow SSH (replace 2222 with your chosen port)
sudo ufw allow 2222/tcp

# Allow HTTP and HTTPS for web services (optional)
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

### Enable UFW

```bash
sudo ufw enable
```

Check the status:

```bash
sudo ufw status
```

> **WARNING**: Always allow your SSH port before enabling UFW, or you’ll lose access to your server.
{:.prompt-danger }

---

## Step 5: Setting Up Basic Backups

Losing data or configurations is a rookie mistake. Start with a simple backup script to save critical files.

```bash
#!/bin/bash
# Create a backup directory with today's date
BACKUP_DIR="/backup/$(date +%Y%m%d)"
mkdir -p $BACKUP_DIR

# Backup system configurations and user data
sudo tar -czf $BACKUP_DIR/etc-backup.tar.gz /etc/
sudo tar -czf $BACKUP_DIR/home-backup.tar.gz /home/

echo "Backup completed: $BACKUP_DIR"
```

Save this as `backup.sh`, make it executable, and run it:

```bash
chmod +x backup.sh
./backup.sh
```

> **TIP**: Follow the 3-2-1 backup rule: 3 copies, 2 different media types, 1 offsite. Consider syncing backups to a cloud service like Backblaze or a separate NAS.
{:.prompt-tip }

---

## Step 6: Installing Docker for Easy Service Management

Docker simplifies running services like media servers or web apps. It’s a great starting point for beginners.

### Install Docker

```bash
# Install Docker and Docker Compose
sudo apt install -y docker.io docker-compose

# Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker

# Add your user to the Docker group
sudo usermod -aG docker username
```

### Test Docker

Run a simple test container:

```bash
docker run hello-world
```

> **INFO**: Log out and back in after adding your user to the Docker group to apply permissions.
{:.prompt-info }

---

## Step 7: Essential Services to Start With

Here are a few beginner-friendly services to get your server doing something useful.

### Pi-hole (Ad Blocker)

```bash
docker run -d --name pihole \
  -p 53:53/tcp -p 53:53/udp -p 80:80 \
  -e TZ="America/New_York" \
  -e WEBPASSWORD="your_secure_password" \
  pihole/pihole
```

### Jellyfin (Media Server)

```bash
docker run -d --name jellyfin \
  -p 8096:8096 \
  -v /path/to/media:/media \
  jellyfin/jellyfin
```

> **TIP**: Access these services via your browser at `http://your_server_ip:port` (e.g., `http://192.168.1.100:8096` for Jellyfin).
{:.prompt-tip }

---

## Common Beginner Mistakes to Avoid

### **Mistake 1: Skipping Updates**

**Problem**: Outdated systems are vulnerable to exploits.  
**Fix**: Schedule regular updates with a cron job:

```bash
# Edit crontab
crontab -e

# Add weekly updates (runs Sundays at 2 AM)
0 2 * * 0 /usr/bin/apt update && /usr/bin/apt upgrade -y
```

### **Mistake 2: Weak Passwords**

**Problem**: Simple passwords are easily cracked.  
**Fix**: Use strong, unique passwords (at least 16 characters) or rely on SSH keys.

### **Mistake 3: No Monitoring**

**Problem**: You won’t know if your server’s struggling until it fails.  
**Fix**: Install `htop` or a monitoring tool like Netdata:

```bash
sudo apt install -y netdata
sudo ufw allow 19999/tcp
```

Access Netdata at `http://your_server_ip:19999`.

---

## Budget and Power Considerations

### Calculate Power Costs

```bash
#!/bin/bash
POWER_WATTS=20
COST_PER_KWH=0.15

POWER_COST=$(echo "scale=2; ($POWER_WATTS/1000) * 8760 * $COST_PER_KWH" | bc)
echo "Annual power cost: $POWER_COST"
```

A mini PC drawing 20W costs about $26/year at $0.15/kWh. Compare this to an old server (150W, ~$200/year).

### Smart Hardware Choices

- **Budget Option ($100-300)**: Used Dell Optiplex or Intel NUC (4-8GB RAM, 120GB SSD).
- **Balanced Option ($500-800)**: Mini PC with 16GB RAM and a small NAS for storage.
- **Avoid**: Old enterprise servers unless you’re okay with high power and noise.

---

## Common Questions Answered

- **“Do I need a static IP?”**  
  Not strictly, but a static internal IP (set in your router) makes life easier. For external access, use dynamic DNS services like DuckDNS.

- **“How do I access my server remotely?”**  
  Set up a VPN (e.g., Tailscale) or use SSH with key-based authentication. Avoid direct port forwarding without a firewall.

- **“What if my server crashes?”**  
  A $50-100 UPS (uninterruptible power supply) ensures graceful shutdowns. Focus on quick recovery with backups.

---

## Tracking Your Progress

### **First Month: The Basics**

- Server updated and secured.
- SSH configured with keys.
- UFW enabled with basic rules.
- First Docker service (e.g., Pi-hole) running.

### **Months 2-3: Expanding**

- Multiple services (e.g., Jellyfin, Nextcloud).
- Automated backups.
- Basic monitoring with Netdata.

### **First Year: Confidence**

- Reverse proxy with Nginx or Traefik.
- VLANs for network segmentation.
- Disaster recovery tested.

---

## When to Ask for Help

- **Stuck for 4+ hours** on the same issue.
- **Confusing errors** or hardware problems.
- **Security concerns** you don’t understand.

### Where to Get Help

- r/linuxadmin or r/homelab on Reddit.
- Ubuntu/Debian forums or Discord servers.
- Stack Overflow for specific error codes.

---

## Keeping It Fun and Manageable

> **INFO**: Your Linux server is a learning tool. If it’s stressing you out, scale back. The goal is to build skills and have fun, not to mimic a data center.
{:.prompt-info }

### Signs You’re Doing Great

- You’re learning new commands and concepts.
- Your services are stable enough for personal use.
- You can recover from small mistakes.
- You’re excited to tinker and try new things.

> **DISCLAIMER**: Hardware and software recommendations are examples based on common setups. Your needs, budget, and environment are unique—always research before buying or deploying.
{:.prompt-info }

Start small, take your time, and enjoy the journey of building your first Linux server. You’ve got this!