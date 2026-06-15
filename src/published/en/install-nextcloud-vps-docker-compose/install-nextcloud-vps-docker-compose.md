---
image: /assets/images/blog/en/install-nextcloud-vps-docker-compose/og-image.png
title: "How to Install Nextcloud on a VPS using Docker Compose"
description: "Learn how to deploy Nextcloud on a Linux VPS using Docker Compose and MariaDB. Build your own private cloud storage without paying monthly fees for Google Drive or iCloud."
date: '2026-06-15'
translationKey: install-nextcloud-vps-docker-compose
locale: en
category: Tutorials
tags:
  - nextcloud
  - docker
  - docker-compose
  - vps
  - cloud-storage
status: published
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - sl0ikkk
  - danielmarszalkowski
howto:
  name: "How to Install Nextcloud on a VPS using Docker Compose"
  totalTime: "PT10M"
  yield: "A fully functional private cloud storage using Nextcloud"
  tool:
    - "A VPS running Linux"
    - "SSH Client (e.g. Terminal, PuTTY)"
    - "Docker and Docker Compose installed"
  steps:
    - name: "Prerequisites"
      text: "Ensure you have Docker and Docker Compose installed."
      url: "step-1-prerequisites"
    - name: "Set up the project"
      text: "Create the project directory and docker-compose.yml file."
      url: "step-2-set-up-the-project"
    - name: "Deploy Nextcloud"
      text: "Spin up the stack and access the web interface."
      url: "step-3-deploy-nextcloud"
faq:
  - question: "What are the system requirements for running Nextcloud in Docker?"
    answer: "You need a VPS with at least 1 GB of RAM (2 GB is recommended for optimal MariaDB database performance) and Docker installed."
  - question: "How do I secure my Nextcloud instance with HTTPS?"
    answer: 'We recommend setting up a reverse proxy like <a href="/blog/setup-nginx-proxy-manager-vps/">Nginx Proxy Manager</a> to point a custom domain with SSL certificates to port 8080.'
  - question: "Can I use PostgreSQL instead of MariaDB?"
    answer: 'Yes, you can modify the <code>docker-compose.yml</code> file to use the official PostgreSQL image instead of MariaDB.'
  - question: "How can I increase the maximum file upload size in Nextcloud?"
    answer: "You can increase it by setting the <code>PHP_UPLOAD_LIMIT</code> and <code>PHP_MEMORY_LIMIT</code> environment variables in your <code>docker-compose.yml</code> file."
  - question: "How do I perform a backup of my self-hosted Nextcloud data?"
    answer: "You should back up both the database volume (using <code>mysqldump</code>) and the persistent <code>nextcloud_data</code> volume where your actual files are stored."
---

## Introduction

Tired of paying monthly fees for Google Drive, Dropbox, or iCloud? With a VPS, you can host your own private, secure cloud storage using **Nextcloud**. Nextcloud is a powerful open-source platform that allows you to store files, sync contacts, and share documents with complete control over your data.

In this guide, we will deploy Nextcloud using **Docker Compose**, ensuring a clean, isolated, and easily manageable installation.

> **Prerequisites:** Before you start, you need a Linux VPS with SSH access, a user account with `sudo` privileges, and Docker with Docker Compose already installed. If you haven't set up Docker Compose yet, check out our [Docker Compose setup guide](/blog/setup-docker-compose/).

## Step 1: Prerequisites

Before starting, ensure your VPS has Docker and Docker Compose installed. If you haven't set them up yet, check out our previous guide on [how to set up Docker Compose](/blog/setup-docker-compose/).

Verify your installation:

{% image "/assets/images/blog/en/install-nextcloud-vps-docker-compose/H1.png", "Running docker compose version to verify installation", "(max-width: 768px) 100vw, 800px" %}

```bash
docker compose version
```

## Step 2: Set up the Project

We need a dedicated directory to store the Nextcloud configuration and persistent data. 

```bash
mkdir ~/nextcloud-server
cd ~/nextcloud-server
```

Now, create the `docker-compose.yml` file:

{% image "/assets/images/blog/en/install-nextcloud-vps-docker-compose/H2.png", "Opening docker-compose.yml with nano editor", "(max-width: 768px) 100vw, 800px" %}

```bash
nano docker-compose.yml
```

Paste the following configuration. This stack includes the official Nextcloud image along with a MariaDB database for optimal performance.

```yaml
services:
  db:
    image: mariadb:latest
    restart: always
    command: --transaction-isolation=READ-COMMITTED --log-bin=binlog --binlog-format=ROW
    volumes:
      - db_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=secure_root_password
      - MYSQL_PASSWORD=secure_db_password
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud

  app:
    image: nextcloud
    restart: always
    ports:
      - 8080:80
    links:
      - db
    volumes:
      - nextcloud_data:/var/www/html
    environment:
      - MYSQL_PASSWORD=secure_db_password
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_HOST=db

volumes:
  db_data:
  nextcloud_data:
```

*Note: Make sure to change `secure_root_password` and `secure_db_password` to strong, unique passwords before saving.*

Save and exit (`CTRL+X`, then `Y`, then `Enter`).

## Step 3: Deploy Nextcloud

With our configuration file ready, it's time to spin up the containers. Run the following command in detached mode:

{% image "/assets/images/blog/en/install-nextcloud-vps-docker-compose/H3.png", "Running docker compose up -d to start the Nextcloud stack", "(max-width: 768px) 100vw, 800px" %}

```bash
docker compose up -d
```

Docker will download the necessary images and start the services. This might take a minute or two depending on your VPS internet speed.

Once the containers are running, open your web browser and navigate to your VPS's IP address on port 8080:

`http://your_server_ip:8080`

{% image "/assets/images/blog/en/install-nextcloud-vps-docker-compose/H4.png", "The Nextcloud setup wizard in the browser", "(max-width: 768px) 100vw, 800px" %}

You will be greeted by the Nextcloud setup wizard. Create your admin account, and Nextcloud will automatically connect to the MariaDB database we configured in the `docker-compose.yml` file.

---

## Conclusion

Congratulations! You now have your own self-hosted Nextcloud instance running securely inside Docker containers on your VPS. Your files are stored privately on hardware you control — with no subscription fees and no third-party access to your data.

As a next step, we strongly recommend placing Nextcloud behind a custom domain with free HTTPS using [Nginx Proxy Manager](/blog/setup-nginx-proxy-manager-vps/), so your cloud is accessible from anywhere via a clean, encrypted URL.

If you need a reliable server to host your self-managed cloud, check out our [Premium VPS plans](/premium-vps/) or [Budget VPS options](/budget-vps/) — both come with fast NVMe SSD storage and high-bandwidth connectivity.