---
image: /assets/images/blog/en/secure-optimize-linux-vps/og-image.png
title: 'Ultimate Guide: Securing and Optimizing Your New Linux VPS'
description: Learn how to properly set up, secure, and optimize your new Linux server after purchase. Includes SSH hardening, firewalls, and performance tuning.
date: '2026-06-17'
translationKey: secure-optimize-linux-vps
locale: en
category: Tutorials
tags:
  - linux
  - vps
  - security
  - ubuntu
status: draft
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - your-github-username
howto:
  name: How to Secure and Optimize a New Linux VPS
  totalTime: PT30M
  yield: A hardened and high-performance Linux server ready for production workloads.
  tool:
    - A VPS running Ubuntu 22.04 or 24.04
    - SSH access with root or sudo privileges
  steps:
    - name: Step 1 — Basic System Information
      text: Check the server layout to understand your hardware resources.
      url: step-1--basic-system-information
    - name: Step 2 — Initial Authentication Setup
      text: Change the hostname and set up a non-root user with sudo permissions.
      url: step-2--initial-authentication-setup
    - name: Step 3 — SSH Hardening
      text: Disable root login and password authentication in favor of SSH keys.
      url: step-3--ssh-hardening
    - name: Step 4 — Network and Firewall Security
      text: Enable UFW, set up CrowdSec, and optimize network with BBR.
      url: step-4--network-and-firewall-security
    - name: Step 5 — Automated Security Updates
      text: Configure unattended-upgrades to keep your system patched automatically.
      url: step-5--automated-security-updates
---

## Introduction

Setting up a new server from a provider like <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> is just the beginning. Often, default images don't follow all security best practices out of the box. To ensure your data is safe and your services run smoothly, you need to perform several critical hardening and optimization steps.

In this guide, we will walk you through the essential process of securing your new [Premium VPS](/premium-vps/) or [Budget VPS](/budget-vps/) from the ground up.

{% image "/assets/images/blog/en/secure-optimize-linux-vps/hero.png", "A secure Linux terminal representing server hardening", "(max-width: 768px) 100vw, 800px" %}

> **Prerequisites:** Before you start, ensure you have access to your server's IP address and root password provided in your <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> dashboard.

---

## Step 1 — Basic System Information

Before making changes, you should understand the hardware and environment you are working with. Run the following commands to get a comprehensive overview of your CPU, memory, disk, and network.

```bash
# Check OS and hardware information
sudo lsb_release -a
sudo lscpu
sudo free -h
sudo lsblk
sudo df -Th
```

Understanding your resource limits helps you decide which services and optimizations are appropriate for your specific plan.

---

## Step 2 — Initial Authentication Setup

### Change Hostname

A proper hostname helps you identify your server in logs and terminal prompts.

```bash
sudo hostnamectl set-hostname your-server-name
```

Update `/etc/hosts` to match:

```bash
# Edit /etc/hosts
127.0.0.1   localhost
127.0.1.1   your-server-name
```

### Create a Secure Non-Root User

Running commands as `root` is dangerous. Create a dedicated user for management tasks.

```bash
sudo adduser your-username
sudo usermod -aG sudo your-username
```

Switch to your new user and verify sudo access:

```bash
su - your-username
sudo whoami
```

---

## Step 3 — SSH Hardening

SSH is the most targeted service on any Linux server. Hardening it is your top priority.

### Deploy SSH Keys

On your **local machine**, generate a key pair if you haven't already:

```bash
ssh-keygen -t ed25519
```

Copy it to your server:

```bash
ssh-copy-id your-username@your-server-ip
```

### Disable Dangerous Defaults

Edit the SSH configuration:

```bash
sudo nano /etc/ssh/sshd_config
```

Set the following parameters:

*   `PermitRootLogin no` — Prevents direct root access.
*   `PasswordAuthentication no` — Forces SSH key usage.
*   `PubkeyAuthentication yes` — Ensures keys are allowed.

Restart the service to apply:

```bash
sudo systemctl restart ssh
```

---

## Step 4 — Network and Firewall Security

### Enable the Firewall (UFW)

By default, all ports might be open. Use Uncomplicated Firewall (UFW) to block everything except what you need.

```bash
sudo ufw allow ssh
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

### Install CrowdSec

CrowdSec is a modern alternative to Fail2Ban that uses community-powered intelligence to block malicious IPs.

```bash
curl -s https://packagecloud.io/install/repositories/crowdsec/crowdsec/script.deb.sh | sudo bash
sudo apt update
sudo apt install crowdsec crowdsec-firewall-bouncer-iptables
```

### Enable BBR for Performance

Google's BBR (Bottleneck Bandwidth and Round-trip propagation time) algorithm can significantly improve network throughput on VPS instances.

```bash
echo 'net.core.default_qdisc=fq' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_congestion_control=bbr' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

---

## Step 5 — Automated Security Updates

Security patches are released almost daily. Don't leave your server vulnerable while you sleep.

```bash
sudo apt update
sudo apt install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

This ensures critical security patches are applied automatically, keeping your [VoxiHost](/premium-vps/) server protected without manual intervention.

---

## Conclusion

By following these steps, you have transformed a default Linux installation into a hardened, high-performance environment. You now have:

*   A secure, key-based authentication system.
*   A restrictive firewall with active threat blocking.
*   Optimized network performance with BBR.
*   Automatic security patching.

Ready for more? Check out our guides on [configuring UFW](/blog/configure-ufw-ubuntu-debian/) or setting up a [LAMP stack](/blog/install-lamp-ubuntu-24-04/).

If you need a powerful server to host your next project, explore our [Premium VPS plans](/premium-vps/) — optimized for performance and reliability.
