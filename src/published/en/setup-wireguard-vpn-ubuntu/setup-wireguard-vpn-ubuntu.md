---
image: /assets/images/blog/setup-wireguard-vpn-ubuntu/og-image.png
title: How to Set Up a WireGuard VPN Server on Ubuntu & Debian
description: A beginner-friendly guide to installing, configuring, and generating client connection keys for a lightning-fast WireGuard VPN server on your Linux VPS.
date: '2026-03-25'
translationKey: setup-wireguard-vpn-ubuntu-debian
category: Tutorials
tags:
  - wireguard
  - vpn
  - ubuntu
  - debian
  - linux
  - vps
  - security
  - networking
howto:
  name: How to Install a WireGuard VPN Server using Angristan's Auto-Install Script
  totalTime: PT10M
  yield: A highly secure, private VPN connection ready for your mobile phone or desktop
  tool:
    - A VPS or dedicated server running Ubuntu or Debian
    - SSH client (e.g. terminal, PuTTY)
    - A user account with sudo privileges
  steps:
    - name: Download the VPN install script
      text: Download the highly trusted Angristan Wireguard script using curl -O https://raw.githubusercontent.com/angristan/wireguard-install/master/wireguard-install.sh
      url: step-1-download-the-trusted-install-script
    - name: Run the autoinstaller
      text: Give the script executable permissions (chmod +x) and run it using sudo ./wireguard-install.sh.
      url: step-2-run-the-auto-installer
    - name: Answer the network prompts
      text: Accept the default networking prompts automatically detected by the script.
      url: step-3-the-configuration-prompts
    - name: Generate your first client connect
      text: Provide a name for your first client (e.g., MyiPhone) to generate the .conf file and QR code.
      url: step-4-generate-your-first-client-key
    - name: Connect your devices
      text: Download the WireGuard App on your phone or PC and scan the QR code to connect securely.
      url: step-5-connect-your-devices
status: published
locale: en
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

**WireGuard** is a modern, revolutionary VPN protocol that has completely dominated the privacy landscape. It is significantly faster, more secure, and connects almost instantaneously without heavily draining cell phone batteries compared to older standards like OpenVPN.

While you *can* manually configure IP tables and NAT forwarding rules to install it, you shouldn't. It is extremely error-prone for beginners.

Instead, the global open-source community relies on a highly audited, universally trusted bash script by developer *Angristan* to seamlessly configure WireGuard on any VPS securely in less than two minutes.

Here is how to deploy your own private VPN on any Linux VPS to bypass restrictions and browse securely.

## Step 1: Download the Trusted Install Script

Log into your server via SSH. Make sure you update your system before executing the installation:

{% image "/assets/images/blog/setup-wireguard-vpn-ubuntu/H1.png", "Running sudo apt update and apt upgrade -y on Ubuntu to update the system before installing WireGuard VPN", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt update && sudo apt upgrade -y
```

Now, download the official installation script directly from Angristan's GitHub repository:

{% image "/assets/images/blog/setup-wireguard-vpn-ubuntu/H2.png", "Downloading the Angristan WireGuard auto-install script from GitHub using curl on Ubuntu VPS", "(max-width: 768px) 100vw, 800px" %}

```bash
curl -O https://raw.githubusercontent.com/angristan/wireguard-install/master/wireguard-install.sh
```

## Step 2: Run the Auto-Installer

Before you can run the file, you **must** tell Linux that this text file is actually an executable script:

{% image "/assets/images/blog/setup-wireguard-vpn-ubuntu/H3.png", "Running sudo chmod and then sudo ./wireguard-install.sh to launch the WireGuard auto-installer on Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo chmod +x wireguard-install.sh
```

Now, execute the script with root privileges:

```bash
sudo ./wireguard-install.sh
```

## Step 3: The Configuration Prompts

{% image "/assets/images/blog/setup-wireguard-vpn-ubuntu/H4.png", "WireGuard install script configuration prompts showing IP address, port, and DNS resolver options on Ubuntu", "(max-width: 768px) 100vw, 800px" %}

The brilliant thing about this script is that it automatically detects your server's network interfaces, public IP addresses, and DNS configurations. 

When you run the script, it will ask you to confirm several settings. **For 99% of deployments, you should simply press `Enter` to accept the default values for every single prompt.**

The prompts will look roughly like this:
```text
IPv4 or IPv6 public address: (Auto-filled with your IP)
Public interface: (Auto-filled, usually eth0 or enp3s0)
WireGuard port: [51820]
First DNS resolver to use for the clients: [1.1.1.1]
```

Press `Enter` through all of them. The script will then rapidly download the `wireguard` packages via `apt`, configure all the complex IP forwarding traffic rules in the Linux kernel, set up the firewall routing, and generate the master server encryption keys.

## Step 4: Generate Your First Client Key

{% image "/assets/images/blog/setup-wireguard-vpn-ubuntu/H5.png", "WireGuard script prompting for a client name and generating the first .conf file and QR code on Ubuntu", "(max-width: 768px) 100vw, 800px" %}

WireGuard uses highly secure peer-to-peer cryptography. To connect your phone or laptop to the VPN, you need to generate a client configuration file (`.conf`) for each device.

Immediately after installing the server packages, the script will automatically prompt you to create your first client:

```text
Client name: 
```
Type a recognizable name without spaces, like `daniel_iphone` or `my_macbook`, and press Enter.

```text
Client's DNS server: [1]
```
Press Enter to accept the default DNS (usually Cloudflare or Google).

The script does something incredibly helpful here. Not only does it create the `.conf` file in your root folder, but it also renders a **massive QR code** directly in your terminal window using ASCII characters!

## Step 5: Connect Your Devices

### Connecting a Mobile Phone (iOS/Android):
Connecting your mobile phone is absurdly simple.
1. Go to the App Store or Google Play Store.
2. Download the official, free **WireGuard** app.
3. Open the app and tap the `+` icon to add a new tunnel.
4. Select **"Create from QR code"**.
5. Point your phone's camera at the giant QR code currently sitting on your computer terminal screen.

Name your connection, flip the toggle switch to "On", and you are immediately encrypting all your mobile traffic through your VPS!

### Connecting a Laptop or PC (Windows/Mac/Linux):
Laptops cannot scan terminal QR codes easily. Instead, you need to retrieve the actual `.conf` file the script generated.

If you named your client `my_macbook`, the script saved a file named `my_macbook.conf` in the directory where you ran the script (usually `/home/youruser/` or `/root/`).

1. Download the `my_macbook.conf` file to your personal computer. (The easiest way to do this securely is using an [SFTP client like FileZilla](/blog/transfer-files-vps-sftp-filezilla/) or WinSCP).
2. Download the official **WireGuard** application for Windows or Mac Desktop from their website.
3. Click "Import tunnel(s) from file" and select `.conf` file.
4. Click "Activate". Your traffic is now secured!

## Generating More Clients

If you want to add a second laptop, a smart TV, or grant secure connection access to a team member, you do not need to reinstall WireGuard. 

Simply run the script again:

{% image "/assets/images/blog/setup-wireguard-vpn-ubuntu/H6.png", "Running the WireGuard install script again to open the management menu for adding more VPN clients", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo ./wireguard-install.sh
```

Because WireGuard is already installed, the script transforms cleanly into a management menu.

Press `1` to instantly generate another `.conf` file and QR code. 

If you want a safe playground to test your WireGuard configuration, **[Budget VPS](/budget-vps/)** plans from **<span class="text-white">Voxi</span><span class="text-amber-300">Host</span>** are a perfect starting point. You can deploy a fresh instance in seconds and start building your private network right away.