---
title: Getting Started with Docker
description: >
  Making sense of containers and how they can make your homelab life easier. Learn Docker basics, common services, and avoid beginner mistakes.
author: greedy
date: 2025-09-29 18:00:00 +1030
categories: [Getting-Started, Docker]
tags: [homelab, beginner, docker, compose, virtualisation]
pin: true
---

If you've been exploring homelabs for any amount of time, you've definitely heard about Docker. It might seem like just another tech buzzword, but once you understand what it does and why it's so useful, you'll wonder how you managed your services without it.

Let's break down what Docker is, why it's perfect for homelabs, and how you can start using it today.

## What is Docker and Why Should You Care?

Think of Docker as a standardized shipping container for software. Instead of worrying about how to install and configure each service individually, Docker packages everything a service needs to run into a neat, self-contained unit called a container.

### The Simple Analogy: Apartments vs Hotels

**Traditional Installation (The Apartment Building)**
- Each service is like an apartment in a building
- They share the same foundation (your operating system)
- If one service has conflicting requirements with another, they can't both live there
- Updates or changes to the building can affect all apartments

**Docker Containers (The Hotel)**
- Each service gets its own private room with its own bathroom and furniture
- Rooms are completely isolated from each other
- Each room can have different requirements without conflict
- You can rearrange or replace rooms without affecting others

## Why Docker is Perfect for Homelabs

### Say Goodbye to "Dependency Hell"

Remember trying to install a service that requires one version of Python, while another service needs a different version? With Docker, each container brings its own dependencies, so conflicts simply don't happen.

### Consistent Environments

A Docker container that runs on your Ubuntu VM will run exactly the same on your friend's Debian server or a Raspberry Pi. No more "but it worked on my machine" problems.

### Easy Backups and Migration

Since everything a service needs is contained in one place, moving it to new hardware or restoring from backup becomes straightforward.

### Quick Experimentation

Want to try out a new service? Instead of going through a complex installation process, you can usually have it running with a single command. Don't like it? Remove it completely without leaving any traces behind.

### Resource Efficiency

Docker containers are much lighter than full virtual machines because they share the host system's kernel. This means you can run more services on the same hardware.

## Docker Basics: Understanding the Pieces

### Images vs Containers

- **Images** are like blueprints - they define what the container will contain
- **Containers** are the actual running instances created from images

### Docker Compose

This is your best friend in the Docker world. Instead of typing long, complex commands, you define your services in a simple YAML file, and Docker Compose handles the rest.

### Volumes

These are how you persist data. When a container is deleted, its file system goes with it. Volumes let you store important data (like databases, configuration files, or media) outside the container.

### Networks

Docker creates virtual networks that your containers can use to communicate with each other and the outside world.

## Installing Docker on Ubuntu

Here's a simple script that will get Docker and Docker Compose installed on your Ubuntu system:

```bash
#!/bin/bash

# Update system and install prerequisites
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Add Docker repository
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# Add your user to the docker group to run without sudo
sudo usermod -aG docker $USER

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Verify installation
docker --version
docker-compose --version

echo "Installation complete! Please log out and back in for group changes to take effect."
```

To use this script:

1. Save it as install-docker.sh
2. Make it executable: chmod +x install-docker.sh
3. Run it: ./install-docker.sh
4. Log out and back in for the group changes to take effect

Security Note: Always review scripts from the internet before running them. This script uses official Docker repositories and methods.
{:.prompt-warning }

Your First Docker Compose Setup

Let's create a simple Docker Compose file that sets up two common homelab services: Portainer (for managing your Docker environment) and Nginx Proxy Manager (for handling web traffic).

Create a file called docker-compose.yml:

```yaml
version: '3.8'

services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./portainer/data:/data
    ports:
      - 9000:9000
    networks:
      - docker-network

  nginx-proxy-manager:
    image: jc21/nginx-proxy-manager:latest
    container_name: nginx-proxy-manager
    restart: unless-stopped
    ports:
      - 80:80
      - 443:443
      - 81:81
    volumes:
      - ./nginx-proxy-manager/data:/data
      - ./nginx-proxy-manager/letsencrypt:/etc/letsencrypt
    networks:
      - docker-network

networks:
  docker-network:
    driver: bridge
```

To get this running:

1. Create a new directory: mkdir ~/docker-services && cd ~/docker-services
2. Save the above content as docker-compose.yml
3. Run: docker-compose up -d

That's it! You now have:

· Portainer running on port 9000 (access via http://your-server-ip:9000)
· Nginx Proxy Manager running on port 81 (access via http://your-server-ip:81)

Use Portainer to visually manage your Docker environment and Nginx Proxy Manager to handle SSL certificates and reverse proxy routing.
{:.prompt-tip }

Common Homelab Services in Docker

Here's a more comprehensive example showing how you might structure multiple services:

```yaml
version: '3.8'

services:
  # Media services
  jellyfin:
    image: jellyfin/jellyfin:latest
    container_name: jellyfin
    restart: unless-stopped
    volumes:
      - ./jellyfin/config:/config
      - ./jellyfin/cache:/cache
      - /path/to/your/media:/media:ro
    ports:
      - 8096:8096
    networks:
      - homelab-network

  # File services
  nextcloud:
    image: nextcloud:latest
    container_name: nextcloud
    restart: unless-stopped
    volumes:
      - ./nextcloud/html:/var/www/html
      - ./nextcloud/apps:/var/www/html/custom_apps
      - ./nextcloud/config:/var/www/html/config
      - ./nextcloud/data:/var/www/html/data
    environment:
      - MYSQL_HOST=mariadb
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=your_secure_password
    depends_on:
      - mariadb
    networks:
      - homelab-network

  # Database for Nextcloud
  mariadb:
    image: mariadb:latest
    container_name: mariadb
    restart: unless-stopped
    volumes:
      - ./mariadb:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=your_secure_root_password
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=your_secure_password
    networks:
      - homelab-network

  # Network services
  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    restart: unless-stopped
    volumes:
      - './pihole/etc-pihole:/etc/pihole'
      - './pihole/etc-dnsmasq.d:/etc/dnsmasq.d'
    environment:
      - TZ=Your/Timezone
      - WEBPASSWORD=your_secure_web_password
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "80:80"
    networks:
      - homelab-network

networks:
  homelab-network:
    driver: bridge
```

Common Mistakes and How to Avoid Them

Mistake 1: Forgetting to Persist Data

The Problem: Storing important data inside containers without volumes, then losing everything when the container is updated or recreated.

The Solution: Always use volumes for:

· Configuration files
· Databases
· User data
· Application data

```yaml
# Good - using volumes
services:
  myapp:
    image: myapp:latest
    volumes:
      - ./app-data:/app/data  # Persists data
      - ./config:/app/config  # Persists configuration
```

Mistake 2: Running as Root

The Problem: Running containers with unnecessary privileges, creating security risks.

The Solution: Use non-root users when possible and read-only volumes where appropriate:

```yaml
services:
  myapp:
    image: myapp:latest
    user: "1000:1000"  # Run as specific user ID
    volumes:
      - ./data:/data:ro  # Read-only volume
```

Mistake 3: Not Using Restart Policies

The Problem: Containers not starting automatically after a reboot or crash.

The Solution: Always set restart policies:

```yaml
services:
  myapp:
    image: myapp:latest
    restart: unless-stopped  # Or "always"
```

Mistake 4: Exposing Too Many Ports

The Problem: Directly exposing every service to the host network.

The Solution: Use a reverse proxy and only expose that:

```yaml
# Instead of this (exposing multiple services):
services:
  service1:
    ports:
      - "8080:80"
  service2:
    ports:
      - "8081:80"

# Do this (only expose reverse proxy):
services:
  nginx-proxy:
    ports:
      - "80:80"
      - "443:443"
  service1:
    # no ports exposed
  service2:
    # no ports exposed
```

Mistake 5: Not Backing Up Volumes

The Problem: Assuming Docker volumes are automatically backed up.

The Solution: Create a backup script for your important volumes:

```bash
#!/bin/bash
# Simple Docker volume backup script
BACKUP_DIR="/backup/$(date +%Y%m%d)"
mkdir -p $BACKUP_DIR

# Backup important volumes
docker run --rm -v your_volume_name:/data -v $BACKUP_DIR:/backup alpine tar czf /backup/your_volume_name.tar.gz -C /data ./

echo "Backups created in: $BACKUP_DIR"
```

Set up automated backups using cron jobs to run this script regularly.
{:.prompt-tip }

Organizing Your Docker Setup

Recommended Directory Structure

```
~/docker-services/
├── docker-compose.yml
├── portainer/
│   └── data/
├── nginx-proxy-manager/
│   ├── data/
│   └── letsencrypt/
├── jellyfin/
│   ├── config/
│   └── cache/
├── nextcloud/
│   ├── html/
│   ├── apps/
│   ├── config/
│   └── data/
└── mariadb/
    └── data/
```

Useful Docker Commands for Daily Use

```bash
# Start all services
docker-compose up -d

# Stop all services
docker-compose down

# View logs for all services
docker-compose logs -f

# View logs for a specific service
docker-compose logs -f service_name

# Check resource usage
docker stats

# Update all containers to latest versions
docker-compose pull
docker-compose up -d

# Backup a specific volume
docker run --rm -v volume_name:/data -v $(pwd):/backup alpine tar czf /backup/backup.tar.gz -C /data ./

# See all running containers
docker ps

# See all containers (including stopped)
docker ps -a

# Execute command inside running container
docker exec -it container_name /bin/bash

# View container resource usage
docker stats container_name
```

When Docker Might Not Be the Right Choice

Consider Traditional Installation When:

· You need maximum performance (some database workloads)
· The service requires specific kernel features
· You're dealing with hardware that needs direct access
· The service doesn't have a well-maintained Docker image

Consider Virtual Machines When:

· You need to run different operating systems (Windows, different Linux distributions)
· You require complete kernel isolation for security
· You're working with services that don't containerize well

Moving Forward

Next Steps After Getting Comfortable

1. Learn about Docker networks for better service isolation
2. Explore Docker Swarm for multi-host setups (the simpler alternative to Kubernetes)
3. Set up monitoring with tools like Portainer or simple bash scripts
4. Implement proper backups for your volumes and configurations
5. Learn about health checks and container orchestration

Recommended Learning Resources

· Docker's official documentation (surprisingly good!)
· The Docker subreddit for specific questions
· Various homelab Discord communities
· Practice by containerizing your own applications

Troubleshooting Common Issues

Container Won't Start

```bash
# Check what's wrong
docker-compose logs service_name

# Check container status
docker ps -a

# Inspect container configuration
docker inspect container_name
```

Port Already in Use

```bash
# Find what's using the port
sudo netstat -tulpn | grep :80

# Or use lsof
sudo lsof -i :80
```

Permission Issues with Volumes

```bash
# Fix volume permissions
sudo chown -R $USER:$USER ./volume_directory
sudo chmod -R 755 ./volume_directory
```

Always read the documentation for the Docker images you're using. Many images have specific requirements or recommended configurations that will save you time and headaches.
{:.prompt-info }

Final Thoughts

Docker might seem intimidating at first, but once you get the basic concepts, it genuinely makes managing homelab services much easier. Start with one or two simple services, get comfortable with the workflow, and gradually expand from there.

Remember that everyone was a beginner at some point, and the Docker community is generally very helpful. Don't be afraid to experiment - that's what homelabs are for!

The best approach is to start simple, make mistakes, learn from them, and gradually build up your knowledge. Before long, you'll be wondering how you ever managed your services without Docker.

Ready to dive deeper? Check out our advanced Docker guide covering networks, security, and orchestration to take your homelab to the next level.