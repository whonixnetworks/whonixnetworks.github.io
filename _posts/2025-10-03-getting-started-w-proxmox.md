---
title: Getting Started with Proxmox
description: >
  Learn why Proxmox is a powerful open-source hypervisor for homelabs, its container and VM management, and best practices for deployment.
author: greedy
date: 2025-10-04 10:00:00 +1030
categories: [Hypervisors, Proxmox]
tags: [homelab, beginner, proxmox, virtualization, lxc]
pin: false
---

![Desktop View](/assets/img/proxmox.png){: .shadow }

If you're looking for an enterprise-grade hypervisor that won't cost you a dime, Proxmox might be exactly what you need. Unlike simpler solutions that prioritize ease of use, Proxmox delivers professional-grade virtualization features in a package that's accessible to homelab enthusiasts.

Let's explore what makes Proxmox powerful and why it's become the go-to choice for serious homelabbers.

## What is Proxmox?

Proxmox Virtual Environment (Proxmox VE) is a complete, open-source server virtualization platform. Think of it as a Swiss Army knife for virtualization:

1. **A Type 1 hypervisor** that runs directly on hardware
2. **A containerization platform** using LXC for lightweight virtualization
3. **A management platform** with a comprehensive web interface
4. **A storage manager** with support for multiple backend technologies

> **Important Note:** Proxmox is open-source and free, with an optional subscription for enterprise support and access to the stable repository.
{: .prompt-info }

## How Proxmox Differs from Other Hypervisors

### Traditional Hypervisors vs Proxmox

| Feature | UNRAID/ESXi | Proxmox |
|---------|-------------|---------|
| **Primary Focus** | Storage/Simplicity | Virtualization & Containers |
| **Cost Model** | One-time fee/Proprietary | Free & Open Source |
| **Container Tech** | Docker | LXC & Docker (in VMs) |
| **Clustering** | Limited | Built-in & Robust |
| **Backup Solution** | Basic | Integrated & Powerful |

### The Virtualization Approach

**Type 2 Hypervisors (VirtualBox, etc.):**
- Run on top of an existing OS
- Performance overhead
- Limited hardware access

**Proxmox's Approach:**
- Bare-metal installation
- Direct hardware access
- Near-native performance
- Professional features like live migration

## Why Proxmox's Architecture Matters

### Real-World Benefits

```bash
# Proxmox lets you run multiple virtualization technologies:
# - KVM for full hardware virtualization
# - LXC for lightweight containerization
# - All managed through a single web interface

# Example deployment:
# VM: Windows 11 (for gaming/work)
# LXC: Ubuntu container (for services)
# LXC: Alpine Linux (for network services)
# All running on the same hardware
```

<u>Benefits for Homelabs</u>

For Learning Enterprise Tech:

· Experience clustering and high availability
· Learn enterprise storage technologies (ZFS, Ceph)
· Practice live migration and backups

For Performance:

· Near-native VM performance
· Lightweight containers for services
· Efficient resource utilization

Proxmox-Specific Terminology

Cluster

Multiple Proxmox nodes working together as a single system, enabling features like live migration and high availability.

LXC (Linux Containers)

Lightweight virtualization technology that shares the host kernel while providing isolated user spaces. Faster than VMs but less isolated.

Storage Backends

Different storage technologies supported by Proxmox:

· ZFS: Advanced filesystem with snapshots and compression
· LVM-Thin: Logical volume manager with thin provisioning
· Ceph: Distributed storage across multiple nodes
· Directory: Simple file-based storage

Templates

Pre-contained system images for quickly deploying containers or VMs.

LXC Containers: The Proxmox Advantage

Why LXC Instead of Docker?

While Proxmox doesn't natively run Docker containers (they run in VMs), LXC provides similar benefits:

```bash
# Docker container:
docker run -d --name nginx -p 80:80 nginx

# Proxmox LXC container:
pct create 100 local:vztmpl/ubuntu-22.04-standard_22.04-1_amd64.tar.zst \
  --storage local-lvm --rootfs 8G --net0 name=eth0,bridge=vmbr0,ip=dhcp
pct start 100
pct exec 100 -- apt update && apt install nginx -y
```

Key LXC Features in Proxmox

Resource Management:

· Direct control over CPU, memory, and storage limits
· Easy resource pool configuration
· Live resource adjustment

Snapshot System:

· Quick container state snapshots
· Easy rollback capabilities
· Integrated with backup system

Network Flexibility:

· Direct network access or NAT
· VLAN support
· Multiple network interfaces per container

Installation and Setup Guide

Hardware Requirements

Minimum:

· 64-bit x86 processor with virtualization support
· 4GB RAM
· 64GB storage
· Network interface

Recommended:

· Intel VT-x or AMD-V capable CPU
· 16GB+ RAM
· SSD for VM storage
· Multiple network interfaces for advanced setups
· ECC RAM for ZFS

Installation Process

1. Download and Create Installer:
   ```bash
   # Download Proxmox ISO from official site
   # Create bootable USB using:
   # - dd (Linux) or Rufus (Windows)
   dd if=proxmox-ve-*.iso of=/dev/sdX bs=1M status=progress
   ```
2. Install and Configure:
   · Boot from USB drive
   · Follow graphical installer
   · Set root password and email
   · Configure network (important for clustering)
   · Select storage disks and filesystem
3. Post-Installation Setup:
   ```bash
   # Access web interface at https://[IP]:8006
   # Accept SSL warning (self-signed certificate)
   # Configure storage via web interface
   # Update system and install latest packages
   apt update && apt dist-upgrade
   ```

Recommended Storage Setup

```yaml
single_node_setup:
  os_disk:
    - "Small SSD (128GB+) for Proxmox OS"
  vm_storage:
    - "Large SSD (500GB+) for VMs and containers"
  backup_storage:
    - "Large HDD for backups and ISO storage"

zfs_configuration:
  filesystem: "ZFS"
  pool_type: "mirror"  # For redundancy
  compression: "lz4"   # Save space
  deduplication: "off" # Usually not worth it for homelabs
```

Best Setup Practices

Network Configuration

Default Setup:

```bash
# Proxmox creates vmbr0 (Linux bridge)
# Typically bridges to your main network interface
# VMs/containers get IPs from your main network

# For advanced setups:
# - Multiple bridges for network segmentation
# - VLANs for isolation
# - Bonding for redundancy
```

Resource Pools

Organization Strategy:

```bash
# Create pools for different purposes:
pvesh create /pools --poolid services
pvesh create /pools --poolid development
pvesh create /pools --poolid production

# Assign VMs/containers to pools
# Set resource limits at pool level
# Monitor usage per pool
```

Backup Strategy

Integrated Backup System:

```bash
# Schedule backups via web interface
# Storage: Local, NFS, or Proxmox Backup Server
# Retention: Keep multiple versions
# Compression: Use zstd for efficiency

# Example backup job:
# - Frequency: Daily
# - Retention: 7 daily, 4 weekly, 12 monthly
# - Mode: Snapshot (for minimal downtime)
```

Common Proxmox Use Cases

The Homelab Powerhouse

```yaml
typical_homelab:
  virtual_machines:
    - "Windows 10/11 (desktop experience)"
    - "Ubuntu Server (Docker host)"
    - "Home Assistant (home automation)"
  
  lxc_containers:
    - "Pi-hole (network-wide ad blocking)"
    - "Nginx Proxy Manager"
    - "WireGuard VPN"
    - "Monitoring (Prometheus + Grafana)"
  
  storage:
    - "ZFS with compression"
    - "Regular snapshot schedule"
    - "Off-site backups"
```

The Learning Environment

```yaml
study_cluster:
  nodes:
    - "pve1 (main node)"
    - "pve2 (secondary node)"
    - "pve3 (tertiary node)"
  
  features:
    - "Live migration between nodes"
    - "High availability for critical services"
    - "Ceph distributed storage"
    - "Cluster-wide resource management"
```

Troubleshooting Common Issues

Network Problems

VM/Container Can't Reach Network:

```bash
# Check bridge configuration:
cat /etc/network/interfaces

# Verify iptables rules:
iptables -L

# Test network from host to guest:
pct exec <CTID> ping 8.8.8.8
```

Storage Issues

Running Out of Space:

```bash
# Check storage usage:
pvesm status

# Clean up old backups and ISOs
# Resize storage if using LVM-Thin
# Consider adding more storage
```

Performance Problems

High CPU/Memory Usage:

```bash
# Check resource usage:
pvesh get /cluster/resources

# Identify problematic VMs/containers
# Adjust resource limits
# Consider moving to more powerful hardware
```

Advanced Features Worth Exploring

Cluster Management

Multi-Node Setup:

```bash
# On first node:
pvecm create CLUSTER_NAME

# On additional nodes:
pvecm add IP_FIRST_NODE

# Benefits:
# - Centralized management
# - Live migration
# - High availability
```

Proxmox Backup Server

Dedicated Backup Solution:

· Separate backup appliance
· Efficient deduplication
· Fast restores
· Integrated with Proxmox VE

ZFS Advanced Features

```bash
# Create a ZFS pool:
zpool create -f tank mirror /dev/sdb /dev/sdc

# Enable compression:
zfs set compression=lz4 tank

# Take snapshots:
zfs snapshot tank/vm-100-disk-0@backup1

# Enable SMB sharing:
zfs set sharesmb=on tank/share
```

Migration Strategy

Moving from Other Virtualization Platforms

From VMware/Proxmox:

1. Use built-in conversion tools
2. Export VMs as OVF/OVA
3. Import into Proxmox
4. Test thoroughly before cutting over

From UNRAID/Type 2 Hypervisors:

1. Create new VMs in Proxmox
2. Use network transfer for data migration
3. Set up containers for services
4. Parallel run for testing

Physical to Virtual (P2V)

Converting Physical Machines:

```bash
# Use Proxmox's built-in P2V tools
# Or use third-party solutions like Clonezilla
# Test virtualized version before decommissioning
```

Cost Analysis

Proxmox vs Alternatives

```bash
# Proxmox Costs:
# - Core software: Free
# - Enterprise repository: €85/year per CPU
# - Community repository: Free

# Compared to:
# - VMware vSphere: $200+/year per CPU
# - Microsoft Hyper-V: Requires Windows Server license
# - UNRAID: $59-$129 one-time fee
```

The free community repository provides timely updates without the enterprise subscription.
{:.prompt-tip}

When Proxmox Might Not Be the Best Choice

Consider Alternatives When

You Want Simple Storage Management:

· UNRAID's drive flexibility is hard to beat
· TrueNAS offers simpler storage-focused management

You Primarily Use Docker:

· While possible via VM, native Docker support is limited
· Consider Docker-centric platforms

You Have Limited Hardware:

· Proxmox works best with proper server hardware
· Older desktop hardware might have compatibility issues

Getting Help and Community

Resources for Beginners

Official Resources:

· Proxmox documentation and wiki
· Official community forum
· Professional training and certification

Community Support:

· Active Reddit community (r/Proxmox)
· Numerous YouTube tutorials
· GitHub repositories with scripts and tools

Learning Path

Recommended Progression:

1. Single node setup with basic VMs
2. LXC container deployment
3. ZFS storage configuration
4. Multi-node clustering
5. Advanced features (Ceph, HA, etc.)

Final Recommendation

Who Should Choose Proxmox?

Choose Proxmox if:

· You want enterprise features without the cost
· You're interested in learning professional virtualization
· You need robust backup and clustering capabilities
· You prefer open-source solutions

Consider Alternatives if:

· You prioritize storage flexibility over virtualization
· You want the simplest possible setup
· Your hardware is very limited or incompatible

Getting Started Checklist

· Download Proxmox ISO
· Verify hardware compatibility
· Plan storage layout
· Install and configure basic system
· Create first VM and LXC container
· Set up backups
· Explore clustering (if multiple nodes)
· Join community forums

Proxmox's professional-grade features combined with its open-source nature make it an exceptional choice for homelabs that want to grow into enterprise-like environments. The learning curve is steeper than some alternatives, but the payoff in capabilities and knowledge gained is substantial.

Remember that while the enterprise subscription offers benefits, the community version is fully functional and receives regular updates.
{:.prompt-info}

Ready to virtualize? The Proxmox community is technical and helpful, with extensive documentation to guide you through even the most complex setups. Start with a single node and expand as your confidence grows.