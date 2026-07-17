---
image: "/assets/images/blog/en/host-valheim-server-ubuntu/og-image.png"
title: "How to Host a Valheim Server on Ubuntu"
description: "Learn how to deploy a private Valheim server on Ubuntu using SteamCMD. This guide covers installation, systemd configuration, and firewall settings."
status: draft
category: "Tutorials"
tags:
  - valheim
  - game-server
  - ubuntu
  - linux
  - steam
  - vps
date: '2026-07-20'
locale: en
translationKey: host-valheim-server-ubuntu
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors: []
howto:
  name: "How to Host a Valheim Server on Ubuntu"
  steps:
    - name: "Step 1: Prepare the System and Dependencies"
      url: "step-1-prepare-the-system-and-dependencies"
    - name: "Step 2: Download and Install the Valheim Server"
      url: "step-2-download-and-install-the-valheim-server"
    - name: "Step 3: Configure the Environment Variables"
      url: "step-3-configure-the-environment-variables"
    - name: "Step 4: Configure the Systemd Service"
      url: "step-4-configure-the-systemd-service"
    - name: "Step 5: Open Firewall Ports and Start the Server"
      url: "step-5-open-firewall-ports-and-start-the-server"
faq:
  - question: "What are the minimum hardware requirements for a Valheim server?"
    answer: "For a smooth experience, we recommend at least <strong>4GB of RAM</strong> and <strong>2 CPU cores</strong>. Our Premium VPS plans are optimized for this workload."
  - question: "How do I update my Valheim dedicated server?"
    answer: "You can update the server by running the <code>steamcmd</code> update command again with the same <code>app_update 896660 validate</code> flag, then restarting the systemd service."
  - question: "Why does my Valheim server fail to start?"
    answer: "Common causes include an incorrect server password length (must be <strong>at least 5 characters</strong>) or conflicting port assignments in the startup script."
  - question: "How can I set an admin password for my Valheim server?"
    answer: "The admin list is managed via the <code>adminlist.txt</code> file, which is generated in the server configuration folder only after the first successful server launch."
  - question: "Do I need to enable cross-play for my server?"
    answer: "Enabling cross-play allows players on different platforms to join your server. Ensure you have ports <strong>2456-2457 UDP</strong> open in the VoxiHost firewall."
---

## Introduction

Valheim has redefined the survival genre with its unique blend of Norse mythology and procedural generation. While playing on public servers is fun, nothing beats the control of hosting your own dedicated instance. By running your own server, you gain full authority over world settings, player count, and persistent availability, ensuring your base remains safe even when you are offline.

For a smooth experience, we recommend using a [Premium VPS](/premium-vps/) with at least 4GB of RAM and two dedicated CPU cores. This hardware overhead ensures that the physics engine and world simulation remain stable as your base grows from a small hut to a sprawling mead hall. Running the server on a [Budget VPS](/budget-vps/) is possible for smaller groups, but keep in mind that Valheim is CPU-intensive during peak combat or terrain modification.

In this guide, we will walk through the entire deployment process on Ubuntu 22.04 LTS. We focus on using SteamCMD for clean installation and systemd for robust process management. By the end of this tutorial, your Valheim world will be live, reachable by your friends, and managed as a native background service that restarts automatically after system reboots.

{% image "/assets/images/blog/en/host-valheim-server-ubuntu/H1.png", "The Valheim main menu screen showing a dedicated server connection", "(max-width: 768px) 100vw, 800px" %}

## Prerequisites

Before beginning the installation, ensure your <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> server instance is running a fresh installation of Ubuntu 22.04 LTS or newer. A clean environment prevents conflicts with existing libraries or conflicting game services. You should have root or sudo access to the terminal, as we will be installing system-level dependencies and configuring a dedicated service account.

Ensure your firewall is prepared to handle the game traffic. Valheim requires UDP ports 2456 through 2457 to be open for both standard play and cross-play connectivity. If you are using UFW, keep these requirements in mind, though we will configure the specific rules later in the guide.

Verify that your server has at least 4GB of RAM available. While the game may launch with less, memory overhead during world generation and entity tracking can lead to severe performance degradation or unexpected crashes. A minimum of 2 CPU cores is essential to handle the simulation load, especially during combat encounters or when multiple players are active in the same area. Ensure your local machine has a reliable SSH client ready to connect to your server instance.

{% image "/assets/images/blog/en/host-valheim-server-ubuntu/H2.png", "Terminal view confirming basic system requirements and SSH access", "(max-width: 768px) 100vw, 800px" %}

## Step 1: Prepare the System and Dependencies

To run a dedicated game server, we must first enable the multiverse repository and ensure the system can handle 32-bit architecture binaries, which are required by the Steam platform. Start by updating your package lists and installing the necessary SteamCMD components.

Run the following commands to configure your repository and pull the required libraries:

```bash
## Enable the multiverse repository for non-free software
sudo add-apt-repository multiverse

## Add support for 32-bit architecture
sudo dpkg --add-architecture i386

## Refresh package lists and install SteamCMD
sudo apt update
sudo apt install -y lib32gcc-s1 steamcmd
```

Once the installation completes, we need to secure the environment by creating a dedicated system user. Running game servers as the root user is a significant security risk. By isolating the process under a specific user account, we limit the potential impact of any vulnerabilities within the server software.

Execute this command to create the user:

```bash
## Create a dedicated system user for Valheim
sudo useradd -m valheim
```

This command creates a new user named `valheim` and automatically generates a home directory at `/home/valheim`. This directory will serve as the base for our server files and configuration. With the dependencies installed and the user account ready, we have established the necessary foundation for the server installation.

{% image "/assets/images/blog/en/host-valheim-server-ubuntu/H3.png", "Terminal showing successful installation of SteamCMD and creation of the valheim user", "(max-width: 768px) 100vw, 800px" %}

## Step 2: Download and Install the Valheim Server

We will now populate the server directory. To avoid permission conflicts and keep the system clean, we will perform the download process directly as our newly created `valheim` user. This ensures that all game files, logs, and configuration data are owned by the correct account, rather than root.

We will use the SteamCMD utility to fetch the dedicated server build. Note that we are specifying the `896660` AppID, which is the official identifier for the Valheim dedicated server. 

Execute the following command to download the server files:

```bash
## Download and validate Valheim server files into the local directory
sudo -u valheim /usr/games/steamcmd +force_install_dir /home/valheim/server +login anonymous +app_update 896660 validate +quit
```

This process may take a few minutes depending on your network speed. The `validate` flag ensures that all files are correctly downloaded and intact. Once the command finishes, you will find the server executable and its supporting libraries located inside `/home/valheim/server`. 

You can confirm the files are present by listing the contents of the directory:

```bash
## Verify the server files exist in the target directory
ls -lh /home/valheim/server
```

You should see a list of files including `valheim_server.x86_64`. With the files successfully downloaded and the ownership correctly assigned to the `valheim` user, we are ready to move on to the service configuration phase.

{% image "/assets/images/blog/en/host-valheim-server-ubuntu/H4.png", "Terminal showing the successful download of Valheim server files via SteamCMD", "(max-width: 768px) 100vw, 800px" %}

## Step 3: Configure the Environment Variables

Valheim requires access to specific Steam libraries to run correctly on Linux. If these paths are not explicitly provided, the binary will fail to launch, reporting missing library errors. To resolve this, we create a script that sets the `LD_LIBRARY_PATH` before invoking the server.

Create a wrapper script in the server directory:

```bash
## Create the wrapper script
sudo -u valheim nano /home/valheim/server/start_valheim.sh
```

Paste the following content into the file:

```bash
#!/bin/bash
export LD_LIBRARY_PATH=./linux64:$LD_LIBRARY_PATH
export SteamAppId=896660
./valheim_server.x86_64 -name "YourServerName" -port 2456 -world "Dedicated" -password "YourPassword" -public 1
```

Save the file and make it executable:

```bash
## Make the script executable
sudo -u valheim chmod +x /home/valheim/server/start_valheim.sh
```

This ensures that whenever the server starts, it correctly loads the dependencies from the `linux64` folder.

{% image "/assets/images/blog/en/host-valheim-server-ubuntu/H5.png", "Terminal showing the created wrapper script", "(max-width: 768px) 100vw, 800px" %}

## Step 4: Configure the Systemd Service

To ensure your server starts automatically on boot and runs reliably in the background, we will create a custom systemd service unit. This approach is superior to running the server in a manual shell session, as it provides built-in restart logic and standard logging capabilities.

Open a new service file using your preferred text editor:

```bash
## Create the systemd service definition file
sudo nano /etc/systemd/system/valheim.service
```

Paste the following configuration into the file. This unit defines the environment, sets the working directory, and executes the wrapper script created in the previous step.

```ini
[Unit]
Description=Valheim Dedicated Server
After=network.target

[Service]
Type=simple
User=valheim
Group=valheim
WorkingDirectory=/home/valheim/server
ExecStart=/home/valheim/server/start_valheim.sh
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

> **Warning:** The server password must be at least 5 characters long. If it is shorter, the Valheim binary will fail to initialize, and the service will enter a crash-loop state.

After saving and closing the file, reload the systemd manager to register the new service.

```bash
## Reload systemd and start the Valheim service
sudo systemctl daemon-reload
sudo systemctl enable --now valheim
```

Your server is now active. You can verify the status at any time by running `sudo systemctl status valheim`.

{% image "/assets/images/blog/en/host-valheim-server-ubuntu/H6.png", "A view of the systemd service file configuration for the Valheim server", "(max-width: 768px) 100vw, 800px" %}

## Step 5: Open Firewall Ports and Start the Server

Valheim requires specific network ports to handle incoming player connections and Steam matchmaking. By default, the game uses the UDP protocol on ports 2456 through 2457. If you are using `ufw` to manage your server security, you must explicitly allow this traffic to reach the game process.

Execute the following commands to configure your firewall rules:

```bash
## Allow the necessary UDP traffic for Valheim
sudo ufw allow 2456:2457/udp
```

If you need to refresh the service to ensure the latest configurations are applied, use the following commands:

```bash
## Restart the service to apply any recent changes
sudo systemctl restart valheim
```

You can confirm that the server is running correctly by checking the system logs. This is especially useful if you encounter connection issues, as it will display the server's initialization progress and show if it has successfully registered with the Steam master server.

```bash
## View the live server logs
sudo journalctl -u valheim -f
```

> **Note:** The server generates several management files, such as `adminlist.txt`, `bannedlist.txt`, and `permittedlist.txt`, only after the first successful launch. If you do not see these files in `/home/valheim/server/`, ensure the service is running and that your user has the necessary write permissions in the directory.

Your server is now reachable. Players can connect using your server's public IP address followed by the port 2456. If you are hosting on a <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Premium VPS](/premium-vps/), ensure your dashboard firewall settings also permit this UDP traffic.

{% image "/assets/images/blog/en/host-valheim-server-ubuntu/H7.png", "Terminal showing the active status of the Valheim systemd service and firewall rules", "(max-width: 768px) 100vw, 800px" %}

## Final Steps

Your dedicated Valheim server is now fully operational on your Ubuntu machine. By offloading the simulation to a <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Premium VPS](/premium-vps/), you ensure consistent performance and low latency for your player base, regardless of your own local connection quality. 

Maintaining your server involves only a few simple habits. To apply game updates, you should stop the service, run the SteamCMD update process, and then bring the service back online. This prevents file corruption during the update window.

```bash
## Stop the server before updating
sudo systemctl stop valheim

## Run the update command
sudo -u valheim /usr/games/steamcmd +force_install_dir /home/valheim/server +login anonymous +app_update 896660 validate +quit

## Start the server again
sudo systemctl start valheim
```

If you notice unexpected behavior or performance drops, always check the logs first using `sudo journalctl -u valheim -n 50`. Most configuration issues stem from incorrect file permissions or firewall rules that conflict with external network restrictions. 

Remember that your world data is stored in the directory specified during your installation. Regular backups of the `/home/valheim/server/` directory are essential to protect your progress against accidental world wipes or configuration errors. With your server live and the backend secured, you are ready to invite your friends to explore the tenth world of Valheim without relying on peer-to-peer hosting limits.

{% image "/assets/images/blog/en/host-valheim-server-ubuntu/H8.png", "Valheim server running successfully on a Linux terminal", "(max-width: 768px) 100vw, 800px" %}
