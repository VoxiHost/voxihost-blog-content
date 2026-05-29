---
image: /assets/images/blog/en/minecraft-1-21-server-centos-rhel/og-image.png
title: How to Setup a Minecraft Vanilla 1.21.1 Server (Java 21) on AlmaLinux, CentOS, Rocky Linux & Fedora
description: Step-by-step guide to installing the latest Minecraft 1.21.1 Vanilla server on AlmaLinux, Rocky Linux, or CentOS using Java 21.
date: '2026-04-23'
translationKey: minecraft-vanilla-java-1-21-server-setup-almalinux-centos-rocky-fedora
locale: en
category: Tutorials
tags:
  - minecraft
  - vanilla
  - java 21
  - server setup
  - almalinux
  - rocky linux
  - centos
howto:
  name: How to Setup a Minecraft Vanilla 1.21+ Server on AlmaLinux, CentOS, Rocky Linux & Fedora
  totalTime: PT10M
  yield: A functional Minecraft 1.21+ Vanilla server powered by Java 21
  tool:
    - AlmaLinux/Rocky VPS
    - Java 21 JRE
    - SSH Client
  steps:
    - name: Environment Preparation
      text: Refresh your package repositories using 'sudo dnf check-update' to prepare for Java installation.
      url: '#step-1-install-java-21'
    - name: Install Java 21
      text: Install the modern Minecraft runtime with 'sudo dnf install java-21-openjdk-headless'.
      url: '#step-1-install-java-21'
    - name: Create a Game User
      text: Setup a restricted 'minecraft' user for security using 'useradd'.
      url: '#step-2-create-a-dedicated-user'
    - name: Download Server Software
      text: Fetch the official Vanilla 1.21.1 server.jar from Mojang's storage.
      url: '#step-3-download-the-server-jar'
    - name: Accept the EULA
      text: Run the JAR once and edit 'eula.txt' to set 'eula=true'.
      url: '#step-4-accept-the-eula'
    - name: Configure Startup
      text: Create a 'start.sh' script with Aikar's optimized RAM flags.
      url: '#step-5-create-launch-script'
    - name: First Launch & OP
      text: Start the server manually to grant yourself administrator privileges.
      url: '#step-6-first-launch-administrator-setup'
    - name: Professional Startup (Systemd)
      text: Set up a systemd service to ensure your server starts automatically on boot.
      url: '#step-7-configure-systemd-service'
status: published
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Setting up a modern Minecraft server requires the modern Java 21 environment. This guide covers **Minecraft 1.20.5 and newer**. If you are looking for other versions, check our [Minecraft Java Server Compatibility Guide](/blog/setup-minecraft-server-centos-rhel/).

### Supported Versions
This Java 21 guide is fully compatible with:
* **1.21 Era:** 1.21.11, 1.21.10, 1.21.9, 1.21.8, 1.21.7, 1.21.6, 1.21.5, 1.21.4, 1.21.3, 1.21.2, 1.21.1, 1.21
* **Early Java 21:** 1.20.6, 1.20.5

For a complete list of direct download URLs for every version, visit our [Minecraft Vanilla Server Download Links Archive](/blog/minecraft-server-download-links/).

At **<span class="text-white">Voxi</span><span class="text-amber-300">Host</span>**, we recommend using at least 6GB of RAM for 1.21+ versions to ensure smooth performance even with high render distances.

> **Security First:** Running a Minecraft server (or any public-facing application) as the `root` user is a major security risk. If the server application is compromised, an attacker would have full access to your entire VPS. Always use a dedicated, restricted user.

## Prerequisites

* A VPS running **AlmaLinux, Rocky Linux, or CentOS** (Available on [Premium VPS](/premium-vps/)).
* Root or sudo access via SSH (for installing Java).
* **A restricted non-root user** to run the server software safely.

## Step 1: Install Java 21

Before installing anything, ensure your system is up-to-date by following our [System Update Guide](/blog/update-centos-rhel/). Once ready, install the headless OpenJDK:

{% image "/assets/images/blog/en/minecraft-1-21-server-centos-rhel/H1.png", "Terminal output showing the installation of OpenJDK 21 on a Linux system", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo dnf check-update
sudo dnf install java-21-openjdk-headless wget -y
```

## Step 2: Create a Dedicated User

For security, never run your server as root. If you are new to Linux permissions, check our guide on [How to Create and Manage Users on AlmaLinux/Rocky](/blog/add-sudo-user-centos/).

{% image "/assets/images/blog/en/minecraft-1-21-server-centos-rhel/H2.png", "Creating a dedicated 'minecraft' user to safely host the 1.21 server", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo useradd -m -r -s /bin/bash minecraft
sudo su - minecraft
mkdir server && cd server
```

## Step 3: Download the Server JAR

Looking for a different modern version? You can find direct Mojang download links for all releases in our [Minecraft Server Download Archive](/blog/minecraft-server-download-links/).

{% image "/assets/images/blog/en/minecraft-1-21-server-centos-rhel/H3.png", "Downloading the official Minecraft 1.21.1 server.jar file from Mojang servers using wget", "(max-width: 768px) 100vw, 800px" %}

Fetch the official Mojang `server.jar` for version 1.21.1:

```bash
wget https://piston-data.mojang.com/v1/objects/59353fb40c36d304f2035d51e7d6e6baa98dc05c/server.jar
```

## Step 4: Accept the EULA

{% image "/assets/images/blog/en/minecraft-1-21-server-centos-rhel/H4.png", "First launch of the 1.21.1 JAR to generate configuration files and accept the EULA", "(max-width: 768px) 100vw, 800px" %}

Run the server once to generate the required configuration files:

```bash
java -jar server.jar nogui
sed -i 's/eula=false/eula=true/' eula.txt
```

## Step 5: Create Launch Script

> **Pro Tip: Using the Nano Editor**
> Nano is a beginner-friendly text editor for the terminal. If the `nano` command is not found, install it using `sudo dnf install nano -y`. 
> * **To Save:** Press `CTRL + O`, then hit `ENTER`.
> * **To Exit:** Press `CTRL + X`.

Create a `start.sh` file to manage your RAM allocation:

{% image "/assets/images/blog/en/minecraft-1-21-server-centos-rhel/H5.png", "Using the nano editor to create and configure the start.sh launch script", "(max-width: 768px) 100vw, 800px" %}

```bash
nano start.sh
```

Paste the following (Aikar's Flags optimized for G1GC):
```bash
#!/bin/bash
java -Xmx6G -Xms6G -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:+AlwaysPreTouch -XX:G1NewSizePercent=30 -XX:G1MaxNewSizePercent=40 -XX:G1HeapRegionSize=8M -XX:G1ReservePercent=20 -XX:G1HeapWastePercent=5 -XX:G1MixedGCCountTarget=4 -XX:InitiatingHeapOccupancyPercent=15 -XX:G1MixedGCLiveThresholdPercent=90 -XX:G1RSetUpdatingPauseTimePercent=5 -XX:SurvivorRatio=32 -XX:+PerfDisableSharedMem -XX:MaxTenuringThreshold=1 -Dusing.aikars.flags=https://mcflags.emc.gs -Daikar.for.v1.20=true -jar server.jar nogui
```

Make it executable:

{% image "/assets/images/blog/en/minecraft-1-21-server-centos-rhel/H6.png", "Setting executable permissions on the start.sh script", "(max-width: 768px) 100vw, 800px" %}
```bash
chmod +x start.sh
```

## Step 6: First Launch & Administrator Setup

Before setting up the automatic background service, you should run the server manually at least once to grant yourself administrator (**OP**) rights.

**1. Start the server manually**

{% image "/assets/images/blog/en/minecraft-1-21-server-centos-rhel/H7.png", "Starting the Minecraft 1.21.1 server manually to access the live console", "(max-width: 768px) 100vw, 800px" %}
Run the launch script you just created:
```bash
./start.sh
```

**2. Grant Administrator (OP) rights**

{% image "/assets/images/blog/en/minecraft-1-21-server-centos-rhel/H8.png", "Granting administrative (OP) privileges via the server console", "(max-width: 768px) 100vw, 800px" %}
Once the server has finished loading (you see the "Done!" message), type your command directly into the console:
```text
op your_minecraft_username
```

**3. Stop the server**

{% image "/assets/images/blog/en/minecraft-1-21-server-centos-rhel/H9.png", "Safely shutting down the Minecraft 1.21.1 server using the stop command", "(max-width: 768px) 100vw, 800px" %}
To save the world data and prepare for background hosting, type:
```text
stop
```
This will return you to the normal Linux command line.

## Step 7: Configure Systemd Service

For a professional setup, we use `systemd`. This ensures your server starts automatically if the VPS reboots and handles crashes gracefully.

Exit the `minecraft` user back to your root/sudo account:
```bash
exit
```

Create the service file:

{% image "/assets/images/blog/en/minecraft-1-21-server-centos-rhel/H10.png", "Creating the minecraft.service systemd file for professional background hosting", "(max-width: 768px) 100vw, 800px" %}
```bash
sudo nano /etc/systemd/system/minecraft.service
```

Paste the following configuration:
```ini
[Unit]
Description=VoxiHost Minecraft 1.21 Server
After=network.target

[Service]
User=minecraft
WorkingDirectory=/home/minecraft/server
ExecStart=/home/minecraft/server/start.sh
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Enable and start your server:

{% image "/assets/images/blog/en/minecraft-1-21-server-centos-rhel/H11.png", "Enabling and starting the minecraft systemd service in the terminal", "(max-width: 768px) 100vw, 800px" %}
```bash
sudo systemctl daemon-reload
sudo systemctl enable minecraft
sudo systemctl start minecraft
```

### Managing Your Server
* **Check Status:** `sudo systemctl status minecraft`
* **View Logs:** `sudo journalctl -u minecraft -f`
* **Stop Server:** `sudo systemctl stop minecraft`

## Next Steps: Security & Management

Now that your server is running, don't forget to:

1. **DDoS Protection**: All <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> servers include automatic [VoxiShield](/shield/) protection. Your server is already being monitored to prevent downtime during attacks.
2. **Open the Firewall**: Allow traffic on port `25565` by running: `sudo firewall-cmd --permanent --add-port=25565/tcp` followed by `sudo firewall-cmd --reload`. For more details, see our [Firewalld Setup Guide](/blog/configure-firewalld-centos-rhel/).
3. **Transfer Files**: Want to upload an existing world? Use SFTP as explained in our [FileZilla Tutorial](/blog/transfer-files-vps-sftp-filezilla/).
4. **Hardening & Monitoring**: Protect your VPS further by [securing SSH](/blog/secure-ssh-centos-rhel/) and [setting up Fail2ban](/blog/setup-fail2ban-centos-rhel/). You can also [monitor your system resources](/blog/monitor-vps-htop-df-free/) using htop.

Now your **Minecraft 1.21.1** server is live! If you need more power for your community, check out our [Premium VPS](/premium-vps/) plans optimized for game hosting.