---
title: Proxmox Cloud Init Guide
description: >
  Learn how to use Cloud Images and Cloud Init with Proxmox to create efficient, optimized VM templates for fast provisioning.
author: greedy
date: 2025-10-03 18:00:00 +1030
categories: [Homelab, Proxmox]
tags: [proxmox, cloud-init, virtualization, automation]
pin: true
---

Using Cloud Images and Cloud Init with Proxmox is easy, fast, efficient, and fun! Cloud Images are small images that are certified cloud ready with Cloud Init preinstalled and ready to accept a Cloud Config. When you combine Cloud Images with Proxmox, you get a perfect, small, efficient, optimized clone template to provision machines with your SSH keys and network settings.

## What You'll Build

By the end of this guide, you'll have:

- A reusable Ubuntu Cloud Image template in Proxmox
- Automated VM provisioning with Cloud Init
- SSH key injection and network configuration
- A fast cloning workflow for new VMs

## Prerequisites

Before starting, ensure you have:

- Proxmox VE 7.0 or newer
- SSH access to your Proxmox server
- Basic familiarity with Linux command line
- Your SSH public key ready

---

## Choose Your Ubuntu Cloud Image

First, select an Ubuntu Cloud Image from the [official Ubuntu Cloud Images](https://cloud-images.ubuntu.com/). For this guide, we'll use Ubuntu 24.04 LTS (Noble Numbat).

## Download the Cloud Image

SSH into your Proxmox server and download the image:

```bash
wget https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img
```

Always verify the checksum of downloaded images for security. You can find checksums on the Ubuntu Cloud Images page.
{:.prompt-tip }

Create a New Virtual Machine

Create a VM with minimal resources (we'll customize this later):

```bash
qm create 8000 --memory 2048 --core 2 --name ubuntu-cloud --net0 virtio,bridge=vmbr0
```

Using VM ID 8000 for templates is a common convention, but you can use any available ID.
{:.prompt-tip }

Import the Disk Image

Import the downloaded disk to your storage:

```bash
qm disk import 8000 noble-server-cloudimg-amd64.img local
```

Replace local with your preferred storage (e.g., local-lvm, nas-storage).

Attach the Disk

Attach the imported disk as a SCSI drive:

```bash
qm set 8000 --scsihw virtio-scsi-pci --scsi0 local:vm-8000-disk-0
```

Add Cloud Init Drive

Configure the Cloud Init drive:

```bash
qm set 8000 --ide2 local:cloudinit
```

Configure Boot Settings

Make the cloud init drive bootable and set boot order:

```bash
qm set 8000 --boot c --bootdisk scsi0
```

Add Serial Console

Enable serial console for better management:

```bash
qm set 8000 --serial0 socket --vga serial0
```

Important: Do not start your VM yet! We need to configure Cloud Init first.
{:.prompt-warning }

Configure Cloud Init

Now let's configure Cloud Init through the Proxmox web interface or CLI:

Via Web Interface:

1. Go to your VM (8000) → Cloud Init
2. Set your SSH public key in the "SSH public keys" field
3. Configure DNS and search domain if needed
4. Set default user (usually ubuntu for Ubuntu images)

Via CLI:

```bash
qm set 8000 --ciuser ubuntu --sshkeys ~/.ssh/authorized_keys
```

Replace the path with your SSH public key file.

Expand Disk Size (Optional)

If you need more disk space, expand it now:

```bash
qm resize 8000 scsi0 +20G
```

This adds 20GB to the base disk. You can adjust this based on your needs.

Create Template

Convert the VM to a template:

```bash
qm template 8000
```

Clone New VMs

Now you can quickly clone new VMs from this template:

```bash
qm clone 8000 135 --name my-new-vm --full
```

Customize the new VM's Cloud Init settings:

```bash
qm set 135 --ciuser ubuntu --ipconfig0 ip=dhcp
# Or for static IP:
qm set 135 --ipconfig0 ip=192.168.1.100/24,gw=192.168.1.1
```

Advanced Cloud Init Configuration

Create a custom Cloud Init configuration file for complex setups:

```yaml
# cloud-init.yaml
#cloud-config
hostname: my-new-vm
manage_etc_hosts: true
users:
  - name: ubuntu
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: users, admin
    home: /home/ubuntu
    shell: /bin/bash
    lock_passwd: false
    ssh-authorized-keys:
      - ssh-rsa AAAAB3NzaC1yc2E...
packages:
  - qemu-guest-agent
  - htop
  - nginx
runcmd:
  - systemctl enable qemu-guest-agent
  - systemctl start qemu-guest-agent
```

Apply the configuration:

```bash
qm set 135 --cicustom "user=local:snippets/cloud-init.yaml"
```

Troubleshooting

Reset Machine ID

If you encounter duplicate machine IDs after cloning:

```bash
sudo rm -f /etc/machine-id
sudo rm -f /var/lib/dbus/machine-id
```

Then shut down the VM. A new ID will be generated on next boot.

If it doesn't regenerate automatically:

```bash
sudo systemd-machine-id-setup
```

Check Cloud Init Logs

If Cloud Init isn't working as expected:

```bash
sudo cat /var/log/cloud-init.log
sudo cat /var/log/cloud-init-output.log
```

Common Issues

Network configuration fails: Ensure your network bridge (vmbr0) is properly configured and the VM has correct IP settings in Cloud Init.
{:.prompt-warning }

SSH keys not injected: Verify the SSH public key format and ensure it's properly set in the Cloud Init configuration.
{:.prompt-warning }

VM won't start after cloning: Check if the source template is stopped and ensure all necessary resources are available.
{:.prompt-warning }

Optimization Tips

Reduce Image Size

Clean up the template before converting:

```bash
sudo apt autoremove -y
sudo apt clean
sudo cloud-init clean
sudo truncate -s 0 /etc/machine-id
```

Enable QEMU Guest Agent

For better integration with Proxmox:

```bash
qm set 8000 --agent enabled=1
```

Set CPU and Memory Limits

Configure resource limits for consistent performance:

```bash
qm set 8000 --cpu host --memory 2048 --balloon 0
```

Complete Setup Script

Here's a complete script to automate the entire process:

```bash
#!/bin/bash

VMID=8000
IMAGE_URL="https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img"
IMAGE_NAME=$(basename $IMAGE_URL)

# Download image
wget $IMAGE_URL

# Create VM
qm create $VMID --memory 2048 --core 2 --name ubuntu-cloud-template --net0 virtio,bridge=vmbr0

# Import disk
qm disk import $VMID $IMAGE_NAME local

# Configure hardware
qm set $VMID --scsihw virtio-scsi-pci --scsi0 local:vm-$VMID-disk-0
qm set $VMID --ide2 local:cloudinit
qm set $VMID --boot c --bootdisk scsi0
qm set $VMID --serial0 socket --vga serial0
qm set $VMID --agent enabled=1

# Expand disk if needed
qm resize $VMID scsi0 +10G

echo "Template $VMID created. Configure Cloud Init settings in the web interface, then run: qm template $VMID"
```

Next Steps

· Create multiple templates for different distributions (Ubuntu, Debian, CentOS)
· Set up automated template updates with cron jobs
· Integrate with configuration management (Ansible, Puppet)
· Explore advanced Cloud Init features like user data scripts
· Set up backup strategies for your templates

With this setup, you can deploy new VMs in seconds rather than minutes, with consistent configurations and your SSH keys automatically injected. Happy virtualizing!