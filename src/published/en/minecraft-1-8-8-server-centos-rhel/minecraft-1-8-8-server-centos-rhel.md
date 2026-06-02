---
image: /assets/images/blog/en/minecraft-1-8-8-server-centos-rhel/og-image.png
title: How to Setup a Classic Minecraft 1.8.8 Server (Java 8) on AlmaLinux, CentOS, Rocky Linux & Fedora
description: A legacy setup guide for Minecraft 1.8.8 servers, perfect for nostalgic PvP and classic Vanilla gameplay using Java 8 on RHEL distributions.
date: '2026-04-23'
updated: '2026-06-02'
translationKey: minecraft-vanilla-java-1-8-8-server-setup-almalinux-centos-rocky-fedora
locale: en
category: Tutorials
tags:
  - minecraft
  - vanilla
  - java 8
  - server setup
  - almalinux
  - rocky linux
  - centos
  - legacy
howto:
  name: How to Setup a Classic Minecraft (1.8.8/1.16.5) Server on AlmaLinux, CentOS, Rocky Linux & Fedora
  totalTime: PT10M
  yield: A functional Classic Minecraft server powered by Java 8
  tool:
    - AlmaLinux/Rocky VPS
    - Java 8 JRE
    - SSH Client
  steps:
    - name: Install Java 8
      text: Install the classic Java environment needed for 1.8.8 through 1.16.5.
      url: step-1-install-java-8
    - name: Create a Game User
      text: Setup a restricted 'minecraft' user for security using 'useradd'.
      url: step-2-create-a-dedicated-user
    - name: Fetch Classic JAR
      text: Download the official 1.8.8 binary safely into your user's home folder.
      url: step-3-download-the-1-8-8-jar
    - name: Legal Terms Agreement
      text: Edit 'eula.txt' to true to signify agreement with Mojang's terms.
      url: step-4-accept-the-eula
    - name: Script Deployment
      text: Create a 'start.sh' file to launch your server with 2GB+ of RAM.
      url: step-5-create-launch-script
    - name: First Launch & OP
      text: Start the server manually to grant yourself administrator privileges.
      url: step-6-first-launch-administrator-setup
    - name: Professional Startup (Systemd)
      text: Set up a systemd service to ensure your server starts automatically on boot.
      url: step-7-configure-systemd-service
faq:
  - question: "Why is Java 8 required for Minecraft 1.8.8?"
    answer: "Legacy Minecraft versions (1.7.10 through 1.16.5) were built and compiled for Java 8. Running them on newer Java runtimes can lead to severe compatibility issues and startup failures."
  - question: "Is Minecraft 1.8.8 safe from the Log4j vulnerability?"
    answer: "No, Minecraft 1.8.8 is vulnerable to Log4Shell by default. You should apply specific startup arguments (like <code>-Dlog4j.configurationFile=...</code>) or use a patched server software like PaperMC to mitigate this risk."
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

Classic Minecraft versions from the **1.7.10 through 1.16.5** era require **Java 8** for legendary stability. This guide covers the entire classic range. For modern version requirements, check our [Minecraft Java Server Compatibility Guide](/blog/setup-minecraft-server-centos-rhel/).

> Legacy versions often have known security vulnerabilities in third-party libraries. NEVER run these versions as root; always use a dedicated, low-privilege user account.

### Supported Versions
This Java 8 guide is fully compatible with:
* **1.16 Era:** 1.16.5, 1.16.4, 1.16.3, 1.16.2, 1.16.1, 1.16
* **1.13 – 1.15:** 1.15.2, 1.15.1, 1.15, 1.14.4, 1.13.2
* **Classic (1.7 – 1.12):** 1.12.2, 1.11.2, 1.10.2, 1.9.4, 1.8.9, 1.8.8, 1.7.10

To visit our [Minecraft Vanilla Server Download Links Archive](/blog/minecraft-server-download-links/), check for the exact link for your version.

## Prerequisites

* A VPS running **AlmaLinux, Rocky Linux, or CentOS** (Available on [Premium VPS](/premium-vps/)).
* Root or sudo access via SSH (for installing Java).
* **A restricted non-root user** to run the server software safely.

## Step 1: Install Java 8

First, perform a full [system update](/blog/update-centos-rhel/) to ensure your package lists are ready.

On standard enterprise servers (AlmaLinux 8 & 9, Rocky Linux 8 & 9), Java 8 is available natively:

{% image "/assets/images/blog/en/minecraft-1-8-8-server-centos-rhel/H1.png", "Terminal output showing the installation of OpenJDK 8 on a Linux server", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo dnf check-update
sudo dnf install java-1.8.0-openjdk-headless wget -y
```

> **Troubleshooting (EL10 & Modern Fedora):** If you receive a `No match for argument` error, it means your Linux distribution is too new and has officially retired Java 8 from its default repositories. You can easily install the highly-optimized **Amazon Corretto 8** distribution by running:
> ```bash
> sudo rpm --import https://yum.corretto.aws/corretto.key
> sudo curl -L -o /etc/yum.repos.d/corretto.repo https://yum.corretto.aws/corretto.repo
> sudo dnf install java-1.8.0-amazon-corretto-devel wget -y
> ```

## Step 2: Create a Dedicated User

For security, never run your server as root. Even legacy versions should be isolated. If you are new to Linux permissions, check our guide on [How to Create and Manage Users on AlmaLinux/Rocky](/blog/add-sudo-user-centos/).

{% image "/assets/images/blog/en/minecraft-1-8-8-server-centos-rhel/H2.png", "Creating a dedicated 'minecraft' user to safely host the legacy server", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo useradd -m -r -s /bin/bash minecraft
sudo su - minecraft
mkdir server && cd server
```

## Step 3: Download the 1.8.8 JAR

Looking for a different classic version? You can find direct Mojang download links for all historical releases in our [Minecraft Server Download Archive](/blog/minecraft-server-download-links/).

{% image "/assets/images/blog/en/minecraft-1-8-8-server-centos-rhel/H3.png", "Downloading the official Minecraft 1.8.8 server.jar file from Mojang servers using wget", "(max-width: 768px) 100vw, 800px" %}

```bash
wget https://launcher.mojang.com/v1/objects/5fafba3f58c40dc51b5c3ca72a98f62dfdae1db7/server.jar
```

## Step 4: Accept the EULA

{% image "/assets/images/blog/en/minecraft-1-8-8-server-centos-rhel/H4.png", "First launch of the 1.8.8 JAR to generate configuration files and accept the EULA", "(max-width: 768px) 100vw, 800px" %}

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

Since 1.8.8 is much lighter than modern versions, 2GB of RAM is often enough for a small group.

{% image "/assets/images/blog/en/minecraft-1-8-8-server-centos-rhel/H5.png", "Using the nano editor to create and configure the start.sh launch script", "(max-width: 768px) 100vw, 800px" %}

```bash
nano start.sh
```

In the editor, paste:
```bash
#!/bin/bash
java -Xmx2G -Xms2G -jar server.jar nogui
```

{% image "/assets/images/blog/en/minecraft-1-8-8-server-centos-rhel/H6.png", "Setting executable permissions on the start.sh script", "(max-width: 768px) 100vw, 800px" %}

Make it executable:
```bash
chmod +x start.sh
```

## Step 6: First Launch & Administrator Setup

Before setting up the automatic background service, you should run the server manually at least once to grant yourself administrator (**OP**) rights.

{% image "/assets/images/blog/en/minecraft-1-8-8-server-centos-rhel/H7.png", "Manually starting the Minecraft 1.8.8 server to access the live console", "(max-width: 768px) 100vw, 800px" %}

**1. Start the server manually**
Run the launch script you just created:
```bash
./start.sh
```

{% image "/assets/images/blog/en/minecraft-1-8-8-server-centos-rhel/H8.png", "Giving yourself administrative (OP) privileges via the server console", "(max-width: 768px) 100vw, 800px" %}

**2. Grant Administrator (OP) rights**
Once the server has finished loading (you see the "Done!" message), type your command directly into the console:
```text
op your_minecraft_username
```

{% image "/assets/images/blog/en/minecraft-1-8-8-server-centos-rhel/H9.png", "Shutting down the Minecraft server safely using the stop command", "(max-width: 768px) 100vw, 800px" %}

**3. Stop the server**
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

{% image "/assets/images/blog/en/minecraft-1-8-8-server-centos-rhel/H10.png", "Creating the minecraft.service systemd file for professional background hosting", "(max-width: 768px) 100vw, 800px" %}

Create the service file:
```bash
sudo nano /etc/systemd/system/minecraft.service
```

Paste the following configuration:
```ini
[Unit]
Description=VoxiHost Minecraft 1.8.8 Server
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

{% image "/assets/images/blog/en/minecraft-1-8-8-server-centos-rhel/H11.png", "Enabling and starting the minecraft systemd service in the terminal", "(max-width: 768px) 100vw, 800px" %}

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

Experience high TPS on our **[Budget VPS](/budget-vps/)** plans!