---
image: /assets/images/blog/en/install-wordpress-docker-compose/og-image.png
title: "How to Install WordPress on a VPS using Docker Compose"
description: "A complete step-by-step guide to deploying a self-hosted WordPress website with a dedicated MySQL database using Docker Compose on a Linux VPS."
date: '2026-06-09'
translationKey: install-wordpress-docker-compose
locale: en
category: Tutorials
tags:
  - wordpress
  - docker
  - docker-compose
  - vps
  - mysql
status: draft
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - sl0ikkk
howto:
  name: "How to Install WordPress on a VPS using Docker Compose"
  totalTime: "PT15M"
  yield: "A server running a complete WordPress site with a MySQL database via Docker Compose"
  tool:
    - "A VPS or dedicated server"
    - "SSH client (e.g. terminal, PuTTY)"
    - "Docker and Docker Compose installed"
  steps:
    - name: "Prepare the Environment"
      text: "Set up the project directory."
      url: "step-1-prepare-the-environment"
    - name: "Create the Configuration"
      text: "Write the docker-compose.yml file."
      url: "step-2-create-the-configuration"
    - name: "Launch and Install"
      text: "Start the stack and run the WordPress installer."
      url: "step-3-launch-and-install"
faq:
  - question: "How do I access my WordPress files on the host system?"
    answer: "All persistent WordPress files (including uploaded media, themes, and plugins) are stored inside the <code>wp_data</code> Docker volume. On standard Linux installations, you can find this volume path in the <code>/var/lib/docker/volumes/</code> directory on your host VPS."
  - question: "How can I change the maximum file upload limit in WordPress when using Docker?"
    answer: "You can increase this limit by creating a custom <code>.user.ini</code> or <code>.htaccess</code> file inside your WordPress volume directory, or by mounting a custom PHP configuration file (e.g. <code>uploads.ini</code> containing <code>upload_max_filesize = 64M</code>) to <code>/usr/local/etc/php/conf.d/uploads.ini</code> in your <code>docker-compose.yml</code>."
  - question: "How do I back up my WordPress Docker container database and files?"
    answer: "To back up the database, run <code>docker compose exec db mysqldump -u wp_user -psecure_wp_password wordpress > backup.sql</code>. To back up files, copy the contents of the <code>wp_data</code> and <code>db_data</code> volumes located in <code>/var/lib/docker/volumes/</code> on your host system."
  - question: "Can I use a custom domain name instead of http://your_server_ip:8080?"
    answer: "Yes. We strongly recommend using a reverse proxy such as Nginx Proxy Manager. You map your domain name to the proxy, which will then securely route traffic from port 80/443 to the WordPress container's port 8080. See our <a href=\"/blog/setup-nginx-proxy-manager-vps/\">Nginx Proxy Manager guide</a> for a complete walkthrough."
  - question: "How do I update WordPress and MySQL to the latest version?"
    answer: "Since both images are set to <code>latest</code> in your <code>docker-compose.yml</code> file, you can update them by running <code>docker compose pull</code> followed by <code>docker compose up -d</code> to recreate the containers with the latest official images."
---

## Introduction

WordPress powers over 40% of the web. While you can install it manually using a LAMP or LEMP stack, using **Docker Compose** is widely considered the modern, superior approach. It isolates your WordPress site and its database into neat containers, making backups, migrations, and updates incredibly simple.

In this tutorial, we will deploy the official WordPress Docker image alongside a dedicated MySQL database.

> **Prerequisites:** Before you start, you need a Linux VPS with SSH access, a user account with `sudo` privileges, and Docker with Docker Compose already installed. See our [Docker Compose setup guide](/blog/setup-docker-compose/) if needed.

## Step 1: Prepare the Environment

Make sure you have Docker and Docker Compose installed on your VPS.

First, let's create a dedicated directory for your new website. This folder will hold your configuration file and all persistent website data (like uploaded images and plugins).

{% image "/assets/images/blog/en/install-wordpress-docker-compose/H1.png", "Creating the wordpress project directory", "(max-width: 768px) 100vw, 800px" %}

```bash
mkdir ~/my-wordpress-site
cd ~/my-wordpress-site
```

## Step 2: Create the Configuration

In Docker Compose, your entire server architecture is defined in a single file.

{% image "/assets/images/blog/en/install-wordpress-docker-compose/H2.png", "Creating docker-compose.yml", "(max-width: 768px) 100vw, 800px" %}

```bash
nano docker-compose.yml
```

Paste the following YAML configuration:

```yaml
services:
  db:
    image: mysql:latest
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: secure_root_password
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wp_user
      MYSQL_PASSWORD: secure_wp_password
    volumes:
      - db_data:/var/lib/mysql

  wordpress:
    image: wordpress:latest
    restart: always
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wp_user
      WORDPRESS_DB_PASSWORD: secure_wp_password
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wp_data:/var/www/html
    depends_on:
      - db

volumes:
  db_data:
  wp_data:
```

### What this file does:
1. **db service**: Pulls the official MySQL Latest image, sets up the database credentials, and mounts a persistent volume `db_data` so your database survives container restarts.
2. **wordpress service**: Pulls the latest WordPress image, maps the internal port 80 to your VPS's public port `8080`, and connects to the database using the credentials we just defined. It also mounts `wp_data` so your themes and plugins are saved to your hard drive.

*Important: Replace `secure_root_password` and `secure_wp_password` with real, strong passwords.*

Save and exit.

## Step 3: Launch and Install

To bring your website online, simply run:

{% image "/assets/images/blog/en/install-wordpress-docker-compose/H3.png", "Running docker compose up -d", "(max-width: 768px) 100vw, 800px" %}

```bash
docker compose up -d
```

Docker will download WordPress and MySQL, link them via an internal network, and start them in the background.

Once the command finishes, open your web browser and navigate to your server's IP address on port 8080:

`http://your_server_ip:8080`

{% image "/assets/images/blog/en/install-wordpress-docker-compose/H4.png", "WordPress Installation Wizard", "(max-width: 768px) 100vw, 800px" %}

You will be greeted by the famous 5-minute WordPress installation screen. Select your language, set your site title, and create your admin account.

---

## Conclusion

Your WordPress site is now live and running inside Docker containers on your VPS. The entire stack — WordPress and MySQL — is defined in a single `docker-compose.yml` file, making it easy to back up, migrate, or replicate to another server at any time.

As a next step, we strongly recommend placing your site behind Nginx Proxy Manager to assign a custom domain name and enable free HTTPS via Let's Encrypt. See our guide on [how to set up Nginx Proxy Manager](/blog/setup-nginx-proxy-manager-vps/) for the complete walkthrough.

If you need a reliable VPS to host your WordPress site, check out our [Premium VPS plans](/premium-vps/) or [Budget VPS options](/budget-vps/) — both come with fast NVMe SSD storage and one-click deployment.
