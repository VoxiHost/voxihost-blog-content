---
image: /assets/images/blog/en/enable-ssh-firewall-ubuntu/og-image.png
title: 'Quick Start: How to Enable SSH and UFW Firewall on Ubuntu'
description: A concise guide on setting up OpenSSH and the Uncomplicated Firewall (UFW) to secure your Linux server for remote access.
date: '2026-06-17'
translationKey: enable-ssh-firewall-ubuntu
locale: en
category: Tutorials
tags:
  - ssh
  - ufw
  - security
  - ubuntu
  - linux
status: draft
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - your-github-username
howto:
  name: Enable SSH and Firewall on Ubuntu
  totalTime: PT5M
  yield: A server accessible via SSH and protected by an active firewall.
  tool:
    - A computer or VPS running Ubuntu
    - Sudo privileges
  steps:
    - name: Step 1 — Install and Enable SSH
      text: Install the OpenSSH server package and start the service.
      url: step-1--install-and-enable-ssh
    - name: Step 2 — Configure and Enable UFW
      text: Allow SSH traffic and activate the Uncomplicated Firewall.
      url: step-2--configure-and-enable-ufw
---

## Introduction

When setting up a new [Budget VPS](/budget-vps/) or local Linux machine, two of the most basic tasks are enabling remote access and securing the network. In this guide, we will show you how to quickly set up the OpenSSH server and the Uncomplicated Firewall (UFW) on Ubuntu.

{% image "/assets/images/blog/en/enable-ssh-firewall-ubuntu/hero.png", "Lock icon next to a terminal prompt", "(max-width: 768px) 100vw, 800px" %}

---

## Step 1 — Install and Enable SSH

By default, SSH might not be installed on desktop versions of Ubuntu. To install and start it, run:

```bash
sudo apt update
sudo apt install openssh-server
```

You can verify that the service is running with:

```bash
sudo systemctl status ssh
```

> **Tip:** If you are using a [Premium VPS](/premium-vps/) from <span class="text-white">Voxi</span><span class="text-amber-300">Host</span>, SSH is usually pre-enabled for you.

---

## Step 2 — Configure and Enable UFW

A firewall is essential for blocking unauthorized connection attempts. Ubuntu comes with UFW, which makes firewall management simple.

First, **crucially**, allow SSH traffic so you don't lock yourself out:

```bash
sudo ufw allow ssh
```

Then, enable the firewall:

```bash
sudo ufw enable
```

You can check which ports are open at any time with:

```bash
sudo ufw status
```

---

## Conclusion

Your server is now ready for remote management and is protected against basic outside intrusion. For more advanced security, we recommend following our guide on [disabling root login and password authentication](/blog/secure-optimize-linux-vps/).

Need a reliable server with 24/7 availability? Explore <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [VPS hosting options](/budget-vps/) starting at competitive prices.
