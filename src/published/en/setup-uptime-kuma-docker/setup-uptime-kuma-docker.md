---
image: /assets/images/blog/en/setup-uptime-kuma-docker/og-image.png
title: "Install Uptime Kuma with Docker: Self-Hosted Monitoring"
description: "Build your own server and website monitoring system using the free Uptime Kuma tool and Docker Compose. Get instant alerts when your services go down."
date: '2026-07-07'
translationKey: "setup-uptime-kuma-docker"
locale: en
category: Tutorials
tags: ["linux", "vps", "docker", "monitoring", "uptime-kuma"]
status: published
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
howto:
  name: "Uptime Kuma Installation"
  totalTime: "PT15M"
  yield: "Your own monitoring server with real-time notifications"
  tool:
    - "A Linux VPS server"
    - "A user account with sudo privileges"
  steps:
    - name: "Step 1: Docker Compose Preparation"
      text: "Installing necessary tools."
      url: "step-1-docker-compose-preparation"
    - name: "Step 2: Uptime Kuma Configuration and Startup"
      text: "Creating the docker-compose.yml file and starting the container."
      url: "step-2-uptime-kuma-configuration-and-startup"
    - name: "Step 3: First Login and Adding a Monitor"
      text: "Adding a server to monitor."
      url: "step-3-first-login-and-adding-a-monitor"
    - name: "Step 4: Configuring Notifications"
      text: "Connecting a Discord Webhook for alerts."
      url: "step-4-configuring-notifications"
faq:
  - question: "Is Uptime Kuma completely free?"
    answer: "Yes, Uptime Kuma is a 100% free and open-source self-hosted monitoring tool under the MIT license."
  - question: "How many resources does Uptime Kuma need?"
    answer: "Uptime Kuma is extremely lightweight and consumes very few resources, making it perfect for running on a low-cost <code>Budget VPS</code>."
  - question: "Can Uptime Kuma monitor Minecraft servers?"
    answer: "Yes, Uptime Kuma has a built-in Minecraft ping feature that queries the game port directly."
---

## Introduction

Have you ever woken up in the morning only to discover that your Minecraft server or website has been down for several hours? Instead of relying on paid solutions like Uptime Robot, you can host your own, beautiful, and 100% free monitoring system.

**Uptime Kuma** is a powerful open-source tool that can monitor websites, game servers, TCP ports, and DNS services. In this quick tutorial, we will show you how to install it on your own server using Docker in just 15 minutes!

> **Tip:** Uptime Kuma does not consume many resources. The cheapest plan from our [Budget VPS](/budget-vps/) lineup (e.g., based on Ubuntu 22.04) will easily be enough for this task.

---

## Step 1: Docker Compose Preparation

The entire Uptime Kuma application runs in a single Docker container. If you don't have Docker installed yet, use the official installation script:

{% image "/assets/images/blog/en/setup-uptime-kuma-docker/H1.png", "Terminal showing Docker installation process", "(max-width: 768px) 100vw, 800px" %}

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo systemctl enable --now docker
```

We recommend using the Docker Compose plugin (it is built-in in the latest versions, but let's make sure):

```bash
sudo apt-get install docker-compose-plugin
```

Check the version to ensure everything is working:

```bash
docker compose version
```

---

## Step 2: Uptime Kuma Configuration and Startup

Now we will create a dedicated folder for our application and configure the YAML file with container settings.

{% image "/assets/images/blog/en/setup-uptime-kuma-docker/H2.png", "Creating docker-compose.yml for Uptime Kuma", "(max-width: 768px) 100vw, 800px" %}

```bash
mkdir -p ~/uptime-kuma
cd ~/uptime-kuma
```

Create a `docker-compose.yml` file using your favorite text editor, for example, `nano`:

```bash
nano docker-compose.yml
```

Paste the following content into it:

```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    volumes:
      - ./uptime-kuma-data:/app/data
    ports:
      - "3001:3001"  # You can change the left value to another port if 3001 is taken
    restart: always
```

Save the file (`Ctrl+O`, `Enter`) and exit the editor (`Ctrl+X`).

Next, run the application in the background using the command:

```bash
sudo docker compose up -d
```

That's it! The application will download the image and start up. You can verify this by typing `sudo docker ps`.

---

## Step 3: First Login and Adding a Monitor

To log into the panel, open your server's IP address in a browser, appending port `3001`, e.g.:
`http://YOUR_IP_ADDRESS:3001`

> **Security:** Remember that while Uptime Kuma forces you to create a password, connecting via HTTP is not the most secure. Be sure to check out our guide on [How to Configure Nginx Proxy Manager](/blog/setup-nginx-proxy-manager-vps/) to connect your domain to Uptime Kuma and generate a free SSL certificate!

Upon entering the site, you will create your administrator account (username and password).

Next, in the top left corner, click **"Add New Monitor"**.
You have a multitude of options to choose from here, e.g.:
- **HTTP(s)**: to check if a website (e.g., your blog) is working.
- **TCP Port**: to check if an SSH process (port 22) or network application is running.
- **Steam Game Server**: ideal for game servers (e.g., FiveM, Rust).
- **Minecraft**: a built-in feature that queries MC servers directly (you provide the IP address and game port).

{% image "/assets/images/blog/en/setup-uptime-kuma-docker/H3.png", "Screen for adding a new monitor for a Minecraft server", "(max-width: 768px) 100vw, 800px" %}

Fill out the form and click "Save". Your first monitor is now up and running!

---

## Step 4: Configuring Notifications

Monitoring is useless if nobody knows about an outage. Uptime Kuma supports over 90(!) notification services, including Telegram, Discord, Slack, Email (SMTP), and even Microsoft Teams.

{% image "/assets/images/blog/en/setup-uptime-kuma-docker/H4.png", "Configuring Discord Webhook notifications in Uptime Kuma", "(max-width: 768px) 100vw, 800px" %}

We will show you how to configure simple notifications on **Discord**:
1. On your Discord server, go to a text channel's Settings -> **Integrations** -> **Create Webhook**. Copy its URL.
2. In Uptime Kuma, go to the top menu and enter **Settings -> Notifications**.
3. Click "Setup Notification".
4. Select type: **Discord**.
5. Paste the copied *Webhook URL*.
6. Click *Test* to see if the bot sent a test message to your channel. If so, save it by clicking **Save**.

Now, when creating or editing any monitor (Step 3), make sure you have selected your newly created notification channel on the right side!

---

## Conclusion

Setting up your own monitoring system is just a few minutes of work. Thanks to Uptime Kuma, you maintain full privacy of your logs and avoid paying subscription fees for premium services (like short, 1-minute check intervals, which can be very expensive in SaaS solutions).

Uptime Kuma installed on a stable <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Budget VPS](/budget-vps/) is a guarantee that you will know about the failure of your main projects before your clients or players notice it!