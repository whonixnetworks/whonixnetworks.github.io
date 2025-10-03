---
title: Getting Started with UNRAID
description: >
  Learn why UNRAID is a unique hypervisor choice for homelabs, its drive flexibility, Docker integration, and best practices for setup.
author: greedy
date: 2025-10-04 10:30:00 +1030
categories: [Hypervisors, Unraid]
tags: [homelab, beginner, unraid, storage, docker]
pin: false
---

If you're looking for a hypervisor that prioritizes storage flexibility and user-friendliness, UNRAID might be your perfect match. Unlike traditional hypervisors that focus purely on virtualization, UNRAID combines storage management, virtualization, and containerization in one elegant package.

Let's explore what makes UNRAID unique and why it's become such a popular choice for media servers and homelabs.

## What is UNRAID?

UNRAID is a network-attached storage (NAS) operating system that also functions as a hypervisor and Docker host. Think of it as three products in one:

1. **A flexible storage server** that lets you mix and match drive sizes
2. **A capable hypervisor** for running virtual machines
3. **A Docker host** with an incredibly user-friendly interface

> **Important Note:** UNRAID is proprietary software with a one-time license fee ($59-$129). There's a 30-day free trial to test it out.
{: .prompt-info }

## How UNRAID Differs from Other Hypervisors

### Traditional Hypervisors vs UNRAID

| Feature | Proxmox/ESXi | UNRAID |
|---------|-------------|--------|
| **Primary Focus** | Virtualization | Storage + Virtualization |
| **Storage** | Requires separate setup | Built-in with flexibility |
| **Cost** | Free (mostly) | One-time license fee |
| **Learning Curve** | Moderate to steep | Gentle |
| **Docker Management** | Manual or via VM | Integrated and visual |

### The Unique Storage Approach

**Traditional RAID:**
- All drives must be the same size
- Limited expansion options
- If one drive fails, you risk total array failure during rebuild

**UNRAID's Approach:**
- Mix any drive sizes you want
- Add drives one at a time as needed
- Single drive failure only affects data on that specific drive
- Two parity drives can protect against dual drive failure

## Why UNRAID's Drive Flexibility Matters

### Real-World Example

```bash
# Traditional RAID requires matching drives:
# - 4TB + 4TB + 4TB + 4TB = 16TB usable (with RAID 5)

# UNRAID lets you mix and match:
# - 4TB (parity) + 8TB + 6TB + 2TB = 16TB usable
# - You can add a 10TB drive later without rebuilding the array
```

Benefits for Homelabs

For Beginners:

· Start with whatever hard drives you have available
· Upgrade gradually as your needs grow
· No need to buy matching drives in bulk

For Media Servers:

· Perfect for Plex/Jellyfin setups with growing media libraries
· Easy to expand storage as your collection grows
· Cost-effective use of leftover drives

UNRAID-Specific Terminology

Array

The main storage pool where your data drives live. This is protected by parity drives.

Parity Drives

Special drives that store recovery information. They must be at least as large as your largest data drive.

Cache Pool

SSD drives used for:

· Docker/VM storage (for speed)
· Temporary file storage before moving to array
· Write acceleration

Shares

Network shares that combine storage from multiple drives into single logical folders.

Docker Containers

Lightweight applications running in isolated environments, managed through UNRAID's web interface.

Docker on UNRAID: Why It's So Popular

The Appstore Experience

UNRAID's Community Applications plugin is like an app store for Docker containers:

```bash
# Instead of command line:
docker run -d --name plex -v /data:/config plexinc/pms-docker

# UNRAID lets you:
# 1. Browse Community Applications
# 2. Search for "Plex"
# 3. Click install
# 4. Fill in a simple form for configuration
```

Key Docker Features in UNRAID

Template System:

· Pre-configured Docker templates for hundreds of applications
· Easy to customize through web forms
· Automatic updates available

Volume Management:

· Visual mapping of container paths to host paths
· Easy backup of appdata
· Integrated with cache and array storage

Network Management:

· Custom Docker networks
· Macvlan for IP-per-container
· Bridge networking made simple

Installation and Setup Guide

Hardware Requirements

Minimum:

· 64-bit x86 processor
· 4GB RAM
· 8GB USB flash drive (for OS)
· At least one storage drive

Recommended:

· Intel CPU with Quick Sync (for media transcoding)
· 16GB+ RAM
· SSD for cache pool
· Multiple drive bays for expansion

Installation Process

1. Create Boot USB:
   ```bash
   # Use UNRAID's USB Creator tool or manually:
   # Download UNRAID ZIP
   # Format USB as FAT32
   # Copy files to USB
   # Run make_bootable script
   ```
2. Boot and Configure:
   · Boot from USB drive
   · Access web interface at http://tower or http://[IP]
   · Start array in "Maintenance Mode"
   · Assign data and parity drives
3. Initial Setup:
   · Set share configurations
   · Configure cache settings
   · Install Community Applications plugin

Recommended Drive Setup

```yaml
initial_setup:
  parity_drives: 
    - "Largest available drive (or two for dual parity)"
  data_drives:
    - "Mix of whatever drives you have"
  cache_pool:
    - "SSD for docker/appdata (250GB+)"
    - "Optional: Second SSD for redundancy"

example_configuration:
  parity_1: "12TB"
  parity_2: "12TB"  # Optional for dual parity
  data_drives: ["8TB", "6TB", "4TB", "2TB"]
  cache: "500GB SSD"
```

Best Setup Practices

Share Configuration

Use Cases for Different Share Types:

```bash
# Media Share (for Plex/Jellyfin)
- Use Cache: "Yes"
- Array: "Array + Cache"
- Split level: "Automatic"

# Appdata Share (Docker configurations)
- Use Cache: "Prefer"
- Array: "Cache"
- Important: Keep this on cache for performance!

# Downloads Share (temporary files)
- Use Cache: "Yes" 
- Array: "Cache + Array"
- mover action: "Array -> Cache"

# Documents Share (important files)
- Use Cache: "Yes"
- Array: "Cache + Array"
- Enable copy-on-write for snapshots
```

Docker Best Practices

Container Organization:

```bash
# Recommended paths:
/mnt/user/appdata/container_name/  # Configurations
/mnt/user/media/                   # Media files  
/mnt/user/backups/                 # Backups
/mnt/user/domains/                 # VM disks
```

Network Setup:

· Use br0 network type for containers needing dedicated IPs
· Use custom networks for container communication
· Consider MACVLAN for advanced networking

Performance Optimization

Cache Strategy:

```bash
# For frequently written data:
Share -> Use Cache -> "Yes"

# For read-heavy, write-once data:
Share -> Use Cache -> "No"

# For Docker/VM performance:
Share -> Use Cache -> "Prefer"
```

Mover Settings:

· Set mover to run during off-hours
· Consider using "Mover Tuning" plugin
· Balance between cache usage and array protection

Common UNRAID Use Cases

The Media Server Powerhouse

```yaml
typical_media_server:
  docker_containers:
    - "Plex or Jellyfin (media playback)"
    - "Sonarr (TV shows)"
    - "Radarr (movies)"
    - "Bazarr (subtitles)"
    - "qBittorrent (downloads)"
  
  shares:
    - "Media (for movies/TV)"
    - "Downloads (incomplete/complete)"
    - "Appdata (container configs)"

  hardware:
    - "Intel CPU with Quick Sync"
    - "16GB+ RAM"
    - "SSD cache for metadata"
    - "Large HDD array for media"
```

The All-in-One Homelab

```yaml
homelab_setup:
  virtual_machines:
    - "Ubuntu Server (for specific services)"
    - "Windows 10/11 (for testing)"
    - "Home Assistant (for automation)"
  
  docker_containers:
    - "Nextcloud (file sync)"
    - "Bitwarden (password manager)"
    - "Nginx Proxy Manager"
    - "Pi-hole (network ad-blocking)"
  
  storage:
    - "Dual parity for data protection"
    - "SSD cache pool for performance"
    - "Multiple shares for organization"
```

Troubleshooting Common Issues

Array Performance

Slow Write Speeds:

```bash
# Check if cache is involved
# Verify share settings
# Check if mover is running
# Consider upgrading cache SSD
```

Docker Problems

Container Won't Start:

```bash
# Check logs in UNRAID web interface
# Verify path mappings
# Check available space on cache
# Review template configuration
```

Drive Issues

Failed Drive:

```bash
# 1. Stop array
# 2. Replace physical drive
# 3. Assign new drive to slot
# 4. Start array to begin rebuild
# 5. Monitor rebuild process
```

Advanced Features Worth Exploring

Plugins Ecosystem

Essential Plugins:

· Community Applications: Docker app store
· Unassigned Devices: Mount external drives
· User Scripts: Schedule custom scripts
· Dynamix File Manager: Web-based file management

VM Management

GPU Passthrough:

· Perfect for gaming VMs or Plex transcoding
· Requires compatible hardware and BIOS settings
· Well-documented in UNRAID community

USB Passthrough:

· Pass specific USB devices to VMs
· Great for USB dongles, security keys, etc.

Migration Strategy

Moving from Another System

From Windows/Linux:

1. Install UNRAID on test hardware first
2. Move data gradually using network transfer
3. Set up Docker containers one by one
4. Test thoroughly before decommissioning old system

From Other NAS:

1. Use UNRAID's "Unassigned Devices" to mount existing drives
2. Copy data to UNRAID array
3. Verify data integrity
4. Repurpose old drives in UNRAID array

Cost Analysis

UNRAID vs Alternatives

```bash
# UNRAID Costs:
# - Basic: $59 (6 storage devices)
# - Plus: $89 (12 storage devices) 
# - Pro: $129 (unlimited storage devices)

# Compared to:
# - FreeNAS/TrueNAS: Free but requires ZFS knowledge
# - Synology: Hardware + software cost
# - Windows Server: Licensing costs
```

The one-time license fee includes all future updates, making it cost-effective long-term.
{:.prompt-tip }

When UNRAID Might Not Be the Best Choice

Consider Alternatives When:

You Need Maximum Performance:

· ZFS (TrueNAS) offers better performance for some workloads
· Traditional RAID can be faster for all-SSD arrays

You Prefer Open Source:

· UNRAID is proprietary software
· TrueNAS Core is fully open source

You Have Matching Drives:

· If you already have identical drives, traditional RAID might make more sense

Getting Help and Community

Resources for Beginners

Official Resources:

· UNRAID documentation and forums
· SpaceInvaderOne YouTube tutorials (highly recommended)
· UNRAID subreddit

Community Support:

· Very active and helpful user community
· Extensive plugin and template library
· Regular feature updates based on user feedback

Final Recommendation

Who Should Choose UNRAID?

Choose UNRAID if:

· You want to mix and match drive sizes
· You prefer a web-based GUI over command line
· You're building a media server or all-in-one homelab
· You value ease of use and gradual learning curve

Consider Alternatives if:

· You need maximum storage performance
· You prefer open-source solutions
· You have a background in enterprise storage
· Budget is extremely constrained

Getting Started Checklist

· Download UNRAID trial
· Gather compatible hardware
· Create boot USB
· Plan your drive layout
· Install Community Applications
· Start with one or two Docker containers
· Explore VM functionality
· Join UNRAID community forums

UNRAID's unique approach to storage management combined with its excellent Docker and VM integration makes it a compelling choice for many homelab users. The ability to start small and grow your storage organically is particularly valuable for beginners.

Remember to use the 30-day trial to ensure UNRAID meets your needs before purchasing a license. The trial includes all features without limitations.
{:.prompt-info }

Ready to dive in? The UNRAID community is one of the most welcoming in the homelab space, and there's no shortage of guides and helpers to get you started.