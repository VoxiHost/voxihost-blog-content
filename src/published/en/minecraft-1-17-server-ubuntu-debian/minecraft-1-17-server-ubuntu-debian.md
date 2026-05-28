---
image: /assets/images/blog/minecraft-1-17-server-ubuntu-debian/og-image.png
title: How to Setup a Minecraft Vanilla 1.17.1 Server (Java 16) on Ubuntu/Debian
description: Instructions for installing a Minecraft 1.17.1 server on Ubuntu/Debian using the specific Java 16 runtime.
date: '2026-04-23'
translationKey: minecraft-vanilla-java-1-17-server-setup-ubuntu-debian
locale: en
category: Tutorials
tags:
  - minecraft
  - vanilla
  - java 16
  - server setup
  - ubuntu
  - debian
howto:
  name: How to Setup a Minecraft Vanilla 1.17 Server on Ubuntu/Debian
  totalTime: PT10M
  yield: A functional Minecraft 1.17 server powered by Java 16
  tool:
    - Ubuntu/Debian VPS
    - Java 16 JRE
    - SSH Client
  steps:
    - name: Verify Repository
      text: Ensure 'sudo apt update' is run to locate the OpenJDK 16 packages.
      url: '#step-1-install-java-16'
    - name: Java 16 Deployment
      text: Install the Java 16 environment required for early 1.17 development.
      url: '#step-1-install-java-16'
    - name: Create a Game User
      text: Setup a restricted 'minecraft' user for security using 'adduser'.
      url: '#step-2-create-a-dedicated-user'
    - name: Fetch server.jar
      text: Download the official 1.17.1 binary from Mojang.
      url: '#step-3-download-1-17-1'
    - name: Approve EULA
      text: Generate and sign the eula.txt file to permit server startup.
      url: '#step-4-run-and-configure'
    - name: Memory Configuration
      text: Deploy a 'start.sh' wrapper with Aikar's optimized RAM flags.
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

Minecraft 1.17.x specifically requires **Java 16**. This guide covers the complete 1.17 lifecycle. For a broader overview of Java requirements, visit our [Minecraft Java Server Compatibility Guide](/blog/setup-minecraft-server-ubuntu-debian/).

> Safety first: Running public game servers as the `root` user exposes your entire system to unnecessary risk. Follow Step 2 carefully to setup a secure environment.

### Supported Versions
This Java 16 guide is fully compatible with:
* **1.17 Era:** 1.17.1, 1.17

To find the exact link for your version, visit our [Minecraft Vanilla Server Download Links Archive](/blog/minecraft-server-download-links/).

## Prerequisites

* A VPS running **Ubuntu or Debian** (Available on [Premium VPS](/premium-vps/)).
* Root or sudo access via SSH (for installing Java).
* **A restricted non-root user** to run the server software safely.

## Step 1: Install Java 16

Before proceeding, we recommend [updating your system](/blog/update-ubuntu-debian/) to ensure stability. 

> **Troubleshooting:** Java 16 is a legacy version and may not be found in the repositories of newer versions (like Ubuntu 24.04+). If you get an "Unable to locate package" error, install **Java 17** instead, it is fully compatible with Minecraft 1.17.1.

**Option A: Install Java 16 (If available)**
```bash
sudo apt update
sudo apt install openjdk-16-jre-headless -y
```

{% image "/assets/images/blog/minecraft-1-17-server-ubuntu-debian/H1.png", "Terminal output showing the installation of OpenJDK on an Ubuntu/Debian system", "(max-width: 768px) 100vw, 800px" %}


**Option B: Install Java 17 (Fallback)**
```bash
sudo apt update
sudo apt install openjdk-17-jre-headless -y
```

## Step 2: Create a Dedicated User

For security, never run your server as root. If you are new to Linux permissions, check our guide on [How to Create and Manage Users on Ubuntu/Debian](/blog/add-sudo-user-ubuntu/).

{% image "/assets/images/blog/minecraft-1-17-server-ubuntu-debian/H2.png", "Creating a dedicated 'minecraft' user for secure server hosting", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo adduser --disabled-password --gecos "" minecraft
sudo su - minecraft
mkdir server && cd server
```

## Step 3: Download 1.17.1

Looking for a different version? You can find direct Mojang download links for all releases in our [Minecraft Server Download Archive](/blog/minecraft-server-download-links/).

{% image "/assets/images/blog/minecraft-1-17-server-ubuntu-debian/H3.png", "Downloading the Minecraft 1.17.1 server.jar from Mojang servers using wget", "(max-width: 768px) 100vw, 800px" %}

```bash
wget https://piston-data.mojang.com/v1/objects/a16d67e5807f57fc4e550299cf20226194497dc2/server.jar
```

## Step 4: Accept the EULA

{% image "/assets/images/blog/minecraft-1-17-server-ubuntu-debian/H4.png", "First launch of the 1.17.1 JAR to generate configuration files and accept the EULA", "(max-width: 768px) 100vw, 800px" %}

Run the server once to generate the required configuration files:

```bash
java -jar server.jar nogui
sed -i 's/eula=false/eula=true/' eula.txt
```

## Step 5: Create Launch Script

> **Pro Tip: Using the Nano Editor**
> Nano is a beginner-friendly text editor for the terminal. If the `nano` command is not found, install it using `sudo apt install nano -y`. 
> * **To Save:** Press `CTRL + O`, then hit `ENTER`.
> * **To Exit:** Press `CTRL + X`.

Paste the following (Aikar's Flags optimized for G1GC):

{% image "/assets/images/blog/minecraft-1-17-server-ubuntu-debian/H5.png", "Using the nano editor to create and configure the start.sh launch script", "(max-width: 768px) 100vw, 800px" %}

```bash
nano start.sh
```

In the editor, paste:
```bash
#!/bin/bash
java -Xmx4G -Xms4G -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:+AlwaysPreTouch -XX:G1NewSizePercent=30 -XX:G1MaxNewSizePercent=40 -XX:G1HeapRegionSize=8M -XX:G1ReservePercent=20 -XX:G1HeapWastePercent=5 -XX:G1MixedGCCountTarget=4 -XX:InitiatingHeapOccupancyPercent=15 -XX:G1MixedGCLiveThresholdPercent=90 -XX:G1RSetUpdatingPauseTimePercent=5 -XX:SurvivorRatio=32 -XX:+PerfDisableSharedMem -XX:MaxTenuringThreshold=1 -Dusing.aikars.flags=https://mcflags.emc.gs -Daikar.for.v1.20=false -jar server.jar nogui
```

{% image "/assets/images/blog/minecraft-1-17-server-ubuntu-debian/H6.png", "Setting executable permissions on the start.sh script", "(max-width: 768px) 100vw, 800px" %}

Make it executable:
```bash
chmod +x start.sh
```

## Step 6: First Launch & Administrator Setup

Before setting up the automatic background service, you should run the server manually at least once to grant yourself administrator (**OP**) rights.

**1. Start the server manually**

{% image "/assets/images/blog/minecraft-1-17-server-ubuntu-debian/H7.png", "Starting the Minecraft 1.17.1 server manually to access the console", "(max-width: 768px) 100vw, 800px" %}

Run the launch script you just created:
```bash
./start.sh
```

**2. Grant Administrator (OP) rights**

{% image "/assets/images/blog/minecraft-1-17-server-ubuntu-debian/H8.png", "Using the op command in the console to grant administrator privileges", "(max-width: 768px) 100vw, 800px" %}

Once the server has finished loading (you see the "Done!" message), type your command directly into the console:
```text
op your_minecraft_username
```

**3. Stop the server**

{% image "/assets/images/blog/minecraft-1-17-server-ubuntu-debian/H9.png", "Executing the stop command to safely shut down the server process", "(max-width: 768px) 100vw, 800px" %}

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

{% image "/assets/images/blog/minecraft-1-17-server-ubuntu-debian/H10.png", "Creating the minecraft.service file for professional background hosting", "(max-width: 768px) 100vw, 800px" %}

Create the service file:
```bash
sudo nano /etc/systemd/system/minecraft.service
```

Paste the following configuration:
```ini
[Unit]
Description=VoxiHost Minecraft 1.17 Server
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

{% image "/assets/images/blog/minecraft-1-17-server-ubuntu-debian/H11.png", "Enabling and starting the minecraft systemd service in terminal", "(max-width: 768px) 100vw, 800px" %}

Enable and start your server:
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
2. **Open the Firewall**: Allow traffic on port `25565` by running: `sudo ufw allow 25565/tcp`. For more details, see our [UFW Setup Guide](/blog/configure-ufw-ubuntu-debian/).
3. **Transfer Files**: Want to upload an existing world? Use SFTP as explained in our [FileZilla Tutorial](/blog/transfer-files-vps-sftp-filezilla/).
4. **Hardening & Monitoring**: Protect your VPS further by [securing SSH](/blog/secure-ssh-ubuntu-debian/) and [setting up Fail2ban](/blog/setup-fail2ban-ubuntu-debian/). You can also [monitor your system resources](/blog/monitor-vps-htop-df-free/) using htop.

Looking for a stable home for your world? Check out **[Premium VPS](/premium-vps/)**.