---
image: /assets/images/blog/en/setup-teamspeak-3-server-ubuntu-debian/og-image.png
title: "How to Set Up a TeamSpeak 3 Server on Ubuntu & Debian"
description: "A complete step-by-step guide to installing, configuring, and securing a TeamSpeak 3 voice server on your Ubuntu or Debian VPS."
date: '2026-06-16'
translationKey: "setup-teamspeak-3-server-ubuntu-debian"
locale: en
category: "Tutorials"
tags: ["teamspeak", "voice server", "ubuntu", "debian"]
status: published
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - sl0ikkk
  - danielmarszalkowski
faq:
  - question: "How do I connect to my TeamSpeak 3 server for the first time?"
    answer: "Open your TeamSpeak 3 client, enter your VPS IP address, and connect. Upon first connection, a dialog will ask for a Privilege Key. Paste the key captured in Step 5 to claim Server Admin permissions."
  - question: "Which ports need to be open in the firewall for TeamSpeak 3?"
    answer: "You need to open port <code>9987/udp</code> for voice traffic, port <code>10011/tcp</code> for ServerQuery, and port <code>30033/tcp</code> for file transfers."
  - question: "How can I recover a lost Privilege Key?"
    answer: "You can generate a new Privilege Key using the ServerQuery interface or through another user who already has administrator privileges. Alternatively, check the log files in the <code>logs/</code> folder for initial keys."
  - question: "Is hosting a TeamSpeak 3 server on a VPS free?"
    answer: "Yes, running the TeamSpeak 3 server software is free for personal, non-commercial use for up to 32 slots without a license. For more slots or commercial usage, you must purchase a license from TeamSpeak."
  - question: "How do I update my TeamSpeak 3 server to the latest version?"
    answer: "Stop the service using <code>sudo systemctl stop teamspeak</code>, download the newest server files from the official website, extract them over your existing installation directory (ensuring you do not overwrite <code>ts3server.sqlitedb</code> or config files), and restart the service."
howto:
  name: "How to Set Up a TeamSpeak 3 Server on Ubuntu & Debian"
  totalTime: "PT15M"
  yield: "A fully working TeamSpeak 3 voice server ready for users"
  tool:
    - "Ubuntu or Debian VPS"
    - "SSH Client"
    - "A user with sudo privileges"
  steps:
    - name: "Step 1: Update the System"
      text: "Ensure all packages are up to date and install required tools like wget and bzip2."
      url: "step-1-update-the-system"
    - name: "Step 2: Create a Dedicated User"
      text: "Create a secure, isolated user for running the TeamSpeak 3 server."
      url: "step-2-create-a-dedicated-user"
    - name: "Step 3: Download and Extract"
      text: "Download the latest TeamSpeak 3 server files and extract them."
      url: "step-3-download-and-extract-teamspeak-3"
    - name: "Step 4: Accept the License"
      text: "Accept the TeamSpeak 3 license agreement to allow the server to run."
      url: "step-4-accept-the-license-agreement"
    - name: "Step 5: First Run & Privilege Key"
      text: "Start the server manually to capture the crucial admin privilege key."
      url: "step-5-first-run-and-privilege-key"
    - name: "Step 6: Systemd Service"
      text: "Create a systemd service to run TeamSpeak automatically in the background."
      url: "step-6-configure-a-systemd-service"
---

TeamSpeak 3 remains one of the most reliable, low-latency, and resource-efficient voice communication platforms for gamers and communities. Self-hosting your own TeamSpeak server gives you ultimate privacy, full control over permissions, and zero reliance on third-party voice chat providers.

In this guide, you will learn how to properly install and secure a TeamSpeak 3 server on your Ubuntu or Debian VPS.

> **Prerequisites:** Before starting, ensure you have [a VPS running Ubuntu or Debian](/premium-vps/) with SSH access and a user account with `sudo` privileges.

## Step 1: Update the System

First, ensure your package index is updated and install the necessary utilities (`wget` for downloading and `bzip2` for extracting the archive):

{% image "/assets/images/blog/en/setup-teamspeak-3-server-ubuntu-debian/H1.png", "Updating packages and installing wget and bzip2 on Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install wget bzip2 -y
```

## Step 2: Create a Dedicated User

Running any public-facing service as the `root` user is a major security risk. Let's create a dedicated system user specifically for TeamSpeak:

{% image "/assets/images/blog/en/setup-teamspeak-3-server-ubuntu-debian/H2.png", "Creating the teamspeak system user", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo adduser --disabled-password --gecos "" teamspeak
```

Now, switch to this new user to perform the installation:

```bash
sudo su - teamspeak
```

## Step 3: Download and Extract TeamSpeak 3

Fetch the latest TeamSpeak 3 server files. You can always find the newest version on the [official TeamSpeak downloads page](https://teamspeak.com/en/downloads/#server).

{% image "/assets/images/blog/en/setup-teamspeak-3-server-ubuntu-debian/H3.png", "Downloading TeamSpeak 3 server archive", "(max-width: 768px) 100vw, 800px" %}

```bash
wget https://files.teamspeak-services.com/releases/server/3.13.8/teamspeak3-server_linux_amd64-3.13.8.tar.bz2
```

Extract the downloaded archive and move the files into the home directory for cleaner organization:

```bash
tar xvf teamspeak3-server_linux_amd64-3.13.8.tar.bz2
mv teamspeak3-server_linux_amd64/* .
rm -rf teamspeak3-server_linux_amd64 teamspeak3-server_linux_amd64-3.13.8.tar.bz2
```

## Step 4: Accept the License Agreement

TeamSpeak requires you to accept their End User License Agreement (EULA) before the server will start. You can do this by creating an empty file named `.ts3server_license_accepted`:

{% image "/assets/images/blog/en/setup-teamspeak-3-server-ubuntu-debian/H4.png", "Accepting the TeamSpeak 3 EULA", "(max-width: 768px) 100vw, 800px" %}

```bash
touch .ts3server_license_accepted
```

## Step 5: First Run and Privilege Key

Now, start the server manually for the first time. **This step is critical** because the server will display your `ServerAdmin` privilege key, which you need to claim admin rights in your TeamSpeak client.

{% image "/assets/images/blog/en/setup-teamspeak-3-server-ubuntu-debian/H5.png", "Starting TS3 to capture the privilege key", "(max-width: 768px) 100vw, 800px" %}

```bash
./ts3server_startscript.sh start
```

You will see output containing your **Privilege Key** and **ServerQuery Admin Credentials**. Copy these details and save them somewhere safe!

Once you have saved the key, stop the server so we can configure it to run as a background service:

```bash
./ts3server_startscript.sh stop
```

## Step 6: Configure a Systemd Service

To ensure your TeamSpeak server starts automatically if your VPS reboots, we need to create a systemd service. First, exit back to your sudo user:

```bash
exit
```

Create a new service file:

{% image "/assets/images/blog/en/setup-teamspeak-3-server-ubuntu-debian/H6.png", "Creating the teamspeak.service file", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo nano /etc/systemd/system/teamspeak.service
```

Paste the following configuration:

```ini
[Unit]
Description=TeamSpeak 3 Server
After=network.target

[Service]
WorkingDirectory=/home/teamspeak
User=teamspeak
Group=teamspeak
Type=forking
ExecStart=/home/teamspeak/ts3server_startscript.sh start inifile=ts3server.ini
ExecStop=/home/teamspeak/ts3server_startscript.sh stop
PIDFile=/home/teamspeak/ts3server.pid
RestartSec=15
Restart=always

[Install]
WantedBy=multi-user.target
```

Save and exit (`CTRL + O`, `ENTER`, `CTRL + X`).

Reload systemd and start your new service:

{% image "/assets/images/blog/en/setup-teamspeak-3-server-ubuntu-debian/H7.png", "Starting the TeamSpeak systemd service", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl daemon-reload
sudo systemctl enable teamspeak
sudo systemctl start teamspeak
```

Check the status to verify it is running perfectly:

```bash
sudo systemctl status teamspeak
```

## Conclusion

Congratulations! You have successfully installed and configured a TeamSpeak 3 server.

To connect, open your TeamSpeak 3 Client, connect to your server's IP address, and paste the **Privilege Key** you saved in Step 5 when prompted. You will instantly be granted Server Admin status.

### Next Steps
* If you have a firewall enabled, ensure you open the correct ports. Check out our guide on [how to configure UFW](/blog/configure-ufw-ubuntu-debian/). TeamSpeak requires ports `9987/udp` (Voice), `10011/tcp` (ServerQuery), and `30033/tcp` (File Transfer).
* Looking for a cost-effective hosting option? Try our [Budget VPS options](/budget-vps/), or get ultimate reliability for large communities on a <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Premium VPS](/premium-vps/) backed by [VoxiShield](/shield/) DDoS Protection.