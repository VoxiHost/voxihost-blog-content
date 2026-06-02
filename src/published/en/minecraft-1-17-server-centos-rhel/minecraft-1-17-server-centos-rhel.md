---
image: /assets/images/blog/en/minecraft-1-17-server-centos-rhel/og-image.png
title: How to Setup a Minecraft Vanilla 1.17.1 Server (Java 16) on AlmaLinux, CentOS, Rocky Linux & Fedora
description: Instructions for installing a Minecraft 1.17.1 server on AlmaLinux, CentOS, or Rocky Linux using the Java 16/17 runtime.
date: '2026-04-23'
translationKey: minecraft-vanilla-java-1-17-server-setup-almalinux-centos-rocky-fedora
locale: en
category: Tutorials
tags:
  - minecraft
  - vanilla
  - java 16
  - java 17
  - server setup
  - almalinux
  - rocky linux
  - centos
howto:
  name: How to Setup a Minecraft Vanilla 1.17 Server on AlmaLinux, CentOS, Rocky Linux & Fedora
  totalTime: PT10M
  yield: A functional Minecraft 1.17 server powered by Java 17 fallback
  tool:
    - AlmaLinux/Rocky VPS
    - Java 17 JRE
    - SSH Client
  steps:
    - name: Java Deployment
      text: Install the Java 17 environment required for 1.17 stability.
      url: step-1-install-java-17-fallback
    - name: Create a Game User
      text: Setup a restricted 'minecraft' user for security using 'useradd'.
      url: step-2-create-a-dedicated-user
    - name: Fetch server.jar
      text: Download the official 1.17.1 binary from Mojang.
      url: step-3-download-1-17-1
    - name: Approve EULA
      text: Generate and sign the eula.txt file to permit server startup.
      url: step-4-accept-the-eula
    - name: Memory Configuration
      text: Deploy a 'start.sh' wrapper with Aikar's optimized RAM flags.
      url: step-5-create-launch-script
    - name: First Launch & OP
      text: Start the server manually to grant yourself administrator privileges.
      url: step-6-first-launch-administrator-setup
    - name: Professional Startup (Systemd)
      text: Set up a systemd service to ensure your server starts automatically on boot.
      url: step-7-configure-systemd-service
faq:
  - question: "Why is Java 16/17 required for Minecraft 1.17?"
    answer: "Minecraft 1.17 was the first version to upgrade its Java requirement from Java 8 to Java 16. Running it on older versions (like Java 8) will prevent the server from starting."
  - question: "Can I use Java 17 instead of Java 16?"
    answer: "Yes, you can run Minecraft 1.17 servers on Java 17 as it is backwards compatible. Since Java 16 is deprecated and not easily available on modern RHEL/AlmaLinux systems, Java 17 is the recommended option."
  - question: "What is the purpose of creating a dedicated 'minecraft' user?"
    answer: "Running the server under a restricted, non-root user account prevents potential security vulnerabilities in Minecraft (or its plugins) from allowing attackers to gain full control of your operating system."
  - question: "How do I allocate more RAM to the server?"
    answer: "You can adjust the allocation by modifying the <code>-Xmx</code> (maximum) and <code>-Xms</code> (starting) memory flags in your launch script. For example, <code>-Xmx4G</code> allocates 4 gigabytes of RAM."
  - question: "How do I make the Minecraft server run automatically when the system boots?"
    answer: "You can create a systemd service file (e.g. <code>/etc/systemd/system/minecraft.service</code>) to manage the server daemon, then enable it using <code>sudo systemctl enable minecraft</code>."
status: published
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Minecraft 1.17.x originally required **Java 16**. However, since Java 16 was a short-lived release, it is rarely available in standard RHEL derivatives (AlmaLinux, Rocky Linux, CentOS 9 Stream). Therefore, we will use **Java 17**, which is fully backwards compatible with 1.17.1. For a broader overview of Java requirements, visit our [Minecraft Java Server Compatibility Guide](/blog/setup-minecraft-server-centos-rhel/).

> Safety first: Running public game servers as the `root` user exposes your entire system to unnecessary risk. Follow Step 2 carefully to setup a secure environment.

### Supported Versions
This guide is fully compatible with:
* **1.17 Era:** 1.17.1, 1.17

To find the exact link for your version, visit our [Minecraft Vanilla Server Download Links Archive](/blog/minecraft-server-download-links/).

## Prerequisites

* A VPS running **AlmaLinux, Rocky Linux, or CentOS** (Available on [Premium VPS](/premium-vps/)).
* Root or sudo access via SSH (for installing Java).
* **A restricted non-root user** to run the server software safely.

## Step 1: Install Java 17 (Fallback)

Before proceeding, we recommend [updating your system](/blog/update-centos-rhel/) to ensure stability. 

As mentioned, Java 16 is a legacy transitional version and is not in the default RHEL `dnf` repositories. Installing the standard LTS **Java 17** is the safest and recommended approach.

```bash
sudo dnf check-update
sudo dnf install java-17-openjdk-headless wget -y
```

{% image "/assets/images/blog/en/minecraft-1-17-server-centos-rhel/H1.png", "Terminal output showing the installation of OpenJDK on a Linux system", "(max-width: 768px) 100vw, 800px" %}

## Step 2: Create a Dedicated User

For security, never run your server as root. If you are new to Linux permissions, check our guide on [How to Create and Manage Users on AlmaLinux/Rocky](/blog/add-sudo-user-centos/).

{% image "/assets/images/blog/en/minecraft-1-17-server-centos-rhel/H2.png", "Creating a dedicated 'minecraft' user for secure server hosting", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo useradd -m -r -s /bin/bash minecraft
sudo su - minecraft
mkdir server && cd server
```

## Step 3: Download 1.17.1

Looking for a different version? You can find direct Mojang download links for all releases in our [Minecraft Server Download Archive](/blog/minecraft-server-download-links/).

{% image "/assets/images/blog/en/minecraft-1-17-server-centos-rhel/H3.png", "Downloading the Minecraft 1.17.1 server.jar from Mojang servers using wget", "(max-width: 768px) 100vw, 800px" %}

```bash
wget https://piston-data.mojang.com/v1/objects/a16d67e5807f57fc4e550299cf20226194497dc2/server.jar
```

## Step 4: Accept the EULA

{% image "/assets/images/blog/en/minecraft-1-17-server-centos-rhel/H4.png", "First launch of the 1.17.1 JAR to generate configuration files and accept the EULA", "(max-width: 768px) 100vw, 800px" %}

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

Paste the following (Aikar's Flags optimized for G1GC):

{% image "/assets/images/blog/en/minecraft-1-17-server-centos-rhel/H5.png", "Using the nano editor to create and configure the start.sh launch script", "(max-width: 768px) 100vw, 800px" %}

```bash
nano start.sh
```

In the editor, paste:
```bash
#!/bin/bash
java -Xmx4G -Xms4G -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:+AlwaysPreTouch -XX:G1NewSizePercent=30 -XX:G1MaxNewSizePercent=40 -XX:G1HeapRegionSize=8M -XX:G1ReservePercent=20 -XX:G1HeapWastePercent=5 -XX:G1MixedGCCountTarget=4 -XX:InitiatingHeapOccupancyPercent=15 -XX:G1MixedGCLiveThresholdPercent=90 -XX:G1RSetUpdatingPauseTimePercent=5 -XX:SurvivorRatio=32 -XX:+PerfDisableSharedMem -XX:MaxTenuringThreshold=1 -Dusing.aikars.flags=https://mcflags.emc.gs -Daikar.for.v1.20=false -jar server.jar nogui
```

{% image "/assets/images/blog/en/minecraft-1-17-server-centos-rhel/H6.png", "Setting executable permissions on the start.sh script", "(max-width: 768px) 100vw, 800px" %}

Make it executable:
```bash
chmod +x start.sh
```

## Step 6: First Launch & Administrator Setup

Before setting up the automatic background service, you should run the server manually at least once to grant yourself administrator (**OP**) rights.

**1. Start the server manually**

{% image "/assets/images/blog/en/minecraft-1-17-server-centos-rhel/H7.png", "Starting the Minecraft 1.17.1 server manually to access the console", "(max-width: 768px) 100vw, 800px" %}

Run the launch script you just created:
```bash
./start.sh
```

**2. Grant Administrator (OP) rights**

{% image "/assets/images/blog/en/minecraft-1-17-server-centos-rhel/H8.png", "Using the op command in the console to grant administrator privileges", "(max-width: 768px) 100vw, 800px" %}

Once the server has finished loading (you see the "Done!" message), type your command directly into the console:
```text
op your_minecraft_username
```

**3. Stop the server**

{% image "/assets/images/blog/en/minecraft-1-17-server-centos-rhel/H9.png", "Executing the stop command to safely shut down the server process", "(max-width: 768px) 100vw, 800px" %}

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

{% image "/assets/images/blog/en/minecraft-1-17-server-centos-rhel/H10.png", "Creating the minecraft.service file for professional background hosting", "(max-width: 768px) 100vw, 800px" %}

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

{% image "/assets/images/blog/en/minecraft-1-17-server-centos-rhel/H11.png", "Enabling and starting the minecraft systemd service in terminal", "(max-width: 768px) 100vw, 800px" %}

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
2. **Open the Firewall**: Allow traffic on port `25565` by running: `sudo firewall-cmd --permanent --add-port=25565/tcp` followed by `sudo firewall-cmd --reload`. For more details, see our [Firewalld Setup Guide](/blog/configure-firewalld-centos-rhel/).
3. **Transfer Files**: Want to upload an existing world? Use SFTP as explained in our [FileZilla Tutorial](/blog/transfer-files-vps-sftp-filezilla/).
4. **Hardening & Monitoring**: Protect your VPS further by [securing SSH](/blog/secure-ssh-centos-rhel/) and [setting up Fail2ban](/blog/setup-fail2ban-centos-rhel/). You can also [monitor your system resources](/blog/monitor-vps-htop-df-free/) using htop.

Looking for a stable home for your world? Check out **[Premium VPS](/premium-vps/)**.