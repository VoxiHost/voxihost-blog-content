---
image: /assets/images/blog/en/minecraft-1-19-server-ubuntu-debian/og-image.png
title: How to Setup a Minecraft Vanilla 1.19.2 Server (Java 17) on Ubuntu/Debian
description: A detailed tutorial on setting up a Minecraft 1.19.2 Vanilla server on Ubuntu using the Java 17 environment.
date: '2026-04-23'
translationKey: minecraft-vanilla-java-1-19-server-setup-ubuntu-debian
locale: en
category: Tutorials
tags:
  - minecraft
  - vanilla
  - java 17
  - server setup
  - ubuntu
  - debian
howto:
  name: How to Setup a Minecraft Vanilla 1.19 Server on Ubuntu/Debian
  totalTime: PT10M
  yield: A functional Minecraft 1.19 era server powered by Java 17
  tool:
    - Ubuntu/Debian VPS
    - Java 17 JRE
    - SSH Client
  steps:
    - name: Java 17 Installation
      text: Install the required LTS Java 17 package via 'sudo apt install openjdk-17-jre-headless'.
      url: step-1-install-java-17
    - name: Create a Game User
      text: Setup a restricted 'minecraft' user for security using 'adduser'.
      url: step-2-create-a-dedicated-user
    - name: Download 1.19.2 JAR
      text: Fetch the official Vanilla 1.19.2 server.jar from Mojang.
      url: step-3-download-minecraft-1-19-2
    - name: EULA Validation
      text: Initialize the server and agree to the EULA by editing 'eula.txt'.
      url: step-4-accept-the-eula
    - name: Start Script Creation
      text: Create a 'start.sh' wrapper with Aikar's optimized RAM flags.
      url: step-5-create-launch-script
    - name: First Launch & OP
      text: Start the server manually to grant yourself administrator privileges.
      url: step-6-first-launch-administrator-setup
    - name: Professional Startup (Systemd)
      text: Set up a systemd service to ensure your server starts automatically on boot.
      url: step-7-configure-systemd-service
faq:
  - question: "Why is Java 17 required for Minecraft 1.19?"
    answer: "Minecraft 1.18 through 1.20.4 require Java 17 to execute correctly. Attempting to run a 1.19 server on older versions (like Java 8) or newer versions (without adjustments) can result in JVM execution errors."
  - question: "What is the purpose of creating a dedicated 'minecraft' user?"
    answer: "Running the server under a restricted, non-root user account prevents potential security vulnerabilities in Minecraft (or its plugins) from allowing attackers to gain full control of your operating system."
  - question: "How do I allocate more RAM to the server?"
    answer: "You can adjust the allocation by modifying the <code>-Xmx</code> (maximum) and <code>-Xms</code> (starting) memory flags in your launch script. For example, <code>-Xmx4G</code> allocates 4 gigabytes of RAM."
  - question: "How do I make the Minecraft server run automatically when the system boots?"
    answer: "You can create a systemd service file (e.g. <code>/etc/systemd/system/minecraft.service</code>) to manage the server daemon, then enable it using <code>sudo systemctl enable minecraft</code>."
  - question: "How do I accept the Minecraft EULA?"
    answer: "When you first launch the server, it will generate a file named <code>eula.txt</code> and exit. Open this file and change the line <code>eula=false</code> to <code>eula=true</code>, then save the file."
status: published
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

The 1.18 to 1.20.4 era of Minecraft relies on **Java 17**. This guide covers the entire Java 17 lifecycle. For information on other versions, see our [Minecraft Java Server Compatibility Guide](/blog/setup-minecraft-server-ubuntu-debian/).

> **Do not run as root:** Always host your Minecraft instance through a restricted user account to protect your VPS files from potential game-layer exploits.

### Prerequisites

* A VPS running **Ubuntu or Debian** (Available on [Premium VPS](/premium-vps/)).
* Sudo access for initial Java installation.
* High-speed SSD/NVMe storage (all <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> nodes use NVMe).

### Supported Versions
This Java 17 guide is fully compatible with:
* **1.20 Era:** 1.20.4, 1.20.3, 1.20.2, 1.20.1, 1.20
* **1.19 Era:** 1.19.4, 1.19.3, 1.19.2, 1.19.1, 1.19
* **1.18 Era:** 1.18.2, 1.18.1, 1.18

Need a different version? Find the direct URL in our [Minecraft Vanilla Server Download Links Archive](/blog/minecraft-server-download-links/).

## Step 1: Install Java 17

Keep your system secure by performing a full [System Update](/blog/update-ubuntu-debian/) first. Then, install the required LTS Java 17 package:

{% image "/assets/images/blog/en/minecraft-1-19-server-ubuntu-debian/H1.png", "Terminal output showing the installation of OpenJDK 17 on an Ubuntu/Debian system", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt update
sudo apt install openjdk-17-jre-headless -y
```

## Step 2: Create a Dedicated User

For security, never run your server as root. If you are new to Linux permissions, check our guide on [How to Create and Manage Users on Ubuntu/Debian](/blog/add-sudo-user-ubuntu/).

{% image "/assets/images/blog/en/minecraft-1-19-server-ubuntu-debian/H2.png", "Creating a dedicated 'minecraft' user to safely host the 1.19 server", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo adduser --disabled-password --gecos "" minecraft
sudo su - minecraft
mkdir server && cd server
```

## Step 3: Download Minecraft 1.19.2

Looking for a different version? You can find direct Mojang download links for all releases in our [Minecraft Server Download Archive](/blog/minecraft-server-download-links/).

{% image "/assets/images/blog/en/minecraft-1-19-server-ubuntu-debian/H3.png", "Downloading the official Minecraft 1.19.2 server.jar file from Mojang servers using wget", "(max-width: 768px) 100vw, 800px" %}

```bash
wget https://piston-data.mojang.com/v1/objects/f69c284232d7c7580bd89a5a4931c3581eae1378/server.jar
```

## Step 4: Accept the EULA

{% image "/assets/images/blog/en/minecraft-1-19-server-ubuntu-debian/H4.png", "First launch of the 1.19.2 JAR to generate configuration files and accept the EULA", "(max-width: 768px) 100vw, 800px" %}

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

{% image "/assets/images/blog/en/minecraft-1-19-server-ubuntu-debian/H5.png", "Using the nano editor to create and configure the start.sh launch script", "(max-width: 768px) 100vw, 800px" %}
```bash
nano start.sh
```

In the editor, paste:
```bash
#!/bin/bash
java -Xmx6G -Xms6G -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:+AlwaysPreTouch -XX:G1NewSizePercent=30 -XX:G1MaxNewSizePercent=40 -XX:G1HeapRegionSize=8M -XX:G1ReservePercent=20 -XX:G1HeapWastePercent=5 -XX:G1MixedGCCountTarget=4 -XX:InitiatingHeapOccupancyPercent=15 -XX:G1MixedGCLiveThresholdPercent=90 -XX:G1RSetUpdatingPauseTimePercent=5 -XX:SurvivorRatio=32 -XX:+PerfDisableSharedMem -XX:MaxTenuringThreshold=1 -Dusing.aikars.flags=https://mcflags.emc.gs -Daikar.for.v1.20=false -jar server.jar nogui
```

Make it executable:

{% image "/assets/images/blog/en/minecraft-1-19-server-ubuntu-debian/H6.png", "Setting executable permissions on the start.sh script", "(max-width: 768px) 100vw, 800px" %}
```bash
chmod +x start.sh
```

## Step 6: First Launch & Administrator Setup

Before setting up the automatic background service, you should run the server manually at least once to grant yourself administrator (**OP**) rights.

**1. Start the server manually**

{% image "/assets/images/blog/en/minecraft-1-19-server-ubuntu-debian/H7.png", "Starting the Minecraft 1.19.2 server manually to access the live console", "(max-width: 768px) 100vw, 800px" %}
Run the launch script you just created:
```bash
./start.sh
```

**2. Grant Administrator (OP) rights**

{% image "/assets/images/blog/en/minecraft-1-19-server-ubuntu-debian/H8.png", "Granting administrative (OP) privileges via the server console", "(max-width: 768px) 100vw, 800px" %}
Once the server has finished loading (you see the "Done!" message), type your command directly into the console:
```text
op your_minecraft_username
```

**3. Stop the server**

{% image "/assets/images/blog/en/minecraft-1-19-server-ubuntu-debian/H9.png", "Safely shutting down the Minecraft 1.19.2 server using the stop command", "(max-width: 768px) 100vw, 800px" %}
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

{% image "/assets/images/blog/en/minecraft-1-19-server-ubuntu-debian/H10.png", "Creating the minecraft.service systemd file for professional background hosting", "(max-width: 768px) 100vw, 800px" %}
```bash
sudo nano /etc/systemd/system/minecraft.service
```

Paste the following configuration:
```ini
[Unit]
Description=VoxiHost Minecraft 1.19 Server
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

{% image "/assets/images/blog/en/minecraft-1-19-server-ubuntu-debian/H11.png", "Enabling and starting the minecraft systemd service in the terminal", "(max-width: 768px) 100vw, 800px" %}
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

Your server is ready! For low-latency hosting, explore our [<span class="text-white">Voxi</span><span class="text-amber-300">Host</span> Budget VPS](/budget-vps/) options.