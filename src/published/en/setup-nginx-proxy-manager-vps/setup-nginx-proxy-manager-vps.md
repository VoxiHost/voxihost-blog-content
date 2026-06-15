---
image: /assets/images/blog/en/setup-nginx-proxy-manager-vps/og-image.png
title: "How to Setup Nginx Proxy Manager on a VPS"
description: "Learn how to effortlessly host multiple websites, map domains, and manage SSL certificates using Nginx Proxy Manager with Docker Compose."
date: '2026-06-02'
updated: '2026-06-02'
translationKey: setup-nginx-proxy-manager-vps
locale: en
category: Tutorials
tags:
  - nginx
  - nginx-proxy-manager
  - docker
  - docker-compose
  - vps
  - ssl
faq:
  - question: "What is a reverse proxy?"
    answer: "A reverse proxy is a server that sits in front of backend applications and forwards client requests to them. It allows you to host multiple applications on a single VPS on different internal ports, routing traffic based on domain names."
  - question: "Why does Nginx Proxy Manager require port 81?"
    answer: "Port 81 is the default port used to access Nginx Proxy Manager's web management interface. Ports 80 and 443 are reserved for public HTTP and HTTPS traffic."
  - question: "How do I secure the web admin panel of Nginx Proxy Manager?"
    answer: "During your first login, NPM will force you to change the default email (<code>admin@example.com</code>) and password (<code>changeme</code>). You can also route port 81 through NPM itself to secure it with an SSL certificate."
  - question: "Can Nginx Proxy Manager handle Let's Encrypt SSL certificates automatically?"
    answer: "Yes. NPM integrates directly with Let's Encrypt. You can request, renew, and assign SSL certificates automatically for any proxy host by toggling the SSL options in the dashboard."
  - question: "What is the difference between Nginx Proxy Manager and raw Nginx?"
    answer: "NPM is a user-friendly wrapper around Nginx that provides a graphical web interface. It generates the under-the-hood Nginx configurations automatically, eliminating the need to write code manually."
status: published
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - sl0ikkk
howto:
  name: "How to Setup Nginx Proxy Manager on a VPS"
  totalTime: "PT15M"
  yield: "A VPS running Nginx Proxy Manager with access to the web management dashboard"
  tool:
    - "VPS or dedicated server (e.g., Ubuntu or Debian)"
    - "SSH Client (e.g., Terminal, PuTTY)"
    - "Docker and Docker Compose installed"
  steps:
    - name: "What is Nginx Proxy Manager?"
      text: "Understand reverse proxies and NPM."
      url: "step-1-what-is-nginx-proxy-manager"
    - name: "Deploying NPM"
      text: "Install NPM via Docker Compose."
      url: "step-2-deploying-npm"
    - name: "Accessing the Panel"
      text: "Log into the beautiful web UI."
      url: "step-3-accessing-the-panel"
---

## Introduction

When you host multiple applications on a single VPS (like WordPress, Nextcloud, and a game panel), you quickly run into a problem: they can't all listen on the default web ports (80 and 443). You end up accessing them via awkward port numbers like `http://your-ip:8080`.

To map real domain names (`cloud.yourdomain.com`) to these apps and secure them with HTTPS, you need a **Reverse Proxy**. **Nginx Proxy Manager (NPM)** is the easiest, most visually appealing way to do this.

> **Prerequisites:** Before you start, you need a VPS running Ubuntu or Debian with SSH access, a user with `sudo` privileges, and Docker with Docker Compose installed. If you need help with Docker Compose, see our [Docker Compose setup guide](/blog/setup-docker-compose/).

## Step 1: What is Nginx Proxy Manager?

Nginx Proxy Manager is a web-based interface that controls Nginx. Instead of writing complex Nginx configuration files by hand, you get a beautiful dashboard where you can route traffic, assign domain names to specific Docker containers, and generate free Let's Encrypt SSL certificates with just a few clicks.

## Step 2: Deploying NPM

Because NPM is containerized, deploying it takes seconds if you have Docker and Docker Compose installed.

Create a new directory for NPM:

{% image "/assets/images/blog/en/setup-nginx-proxy-manager-vps/H1.png", "Creating the npm directory", "(max-width: 768px) 100vw, 800px" %}

```bash
mkdir ~/npm-server
cd ~/npm-server
```

Create your `docker-compose.yml` file:

{% image "/assets/images/blog/en/setup-nginx-proxy-manager-vps/H2.png", "Editing docker-compose.yml", "(max-width: 768px) 100vw, 800px" %}

```bash
nano docker-compose.yml
```

Paste the official recommended stack:

```yaml
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      # These ports are in format <host-port>:<container-port>
      - '80:80' # Public HTTP Port
      - '81:81' # Admin Web Port
      - '443:443' # Public HTTPS Port
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```

Save and exit. Notice how NPM takes over ports `80` and `443` for your entire VPS. It will intercept all incoming web traffic and seamlessly route it to the correct internal container!

Spin up the manager:

{% image "/assets/images/blog/en/setup-nginx-proxy-manager-vps/H3.png", "Starting Nginx Proxy Manager", "(max-width: 768px) 100vw, 800px" %}

```bash
docker compose up -d
```

## Step 3: Accessing the Panel

Once the containers are running, you can access the admin panel by visiting your server's IP address on port `81`.

`http://your_server_ip:81`

{% image "/assets/images/blog/en/setup-nginx-proxy-manager-vps/H4.png", "Nginx Proxy Manager Login Screen", "(max-width: 768px) 100vw, 800px" %}

**Default Login Credentials:**
- Email: `admin@example.com`
- Password: `changeme`

Immediately upon logging in, NPM will prompt you to change these default credentials. Do so to secure your proxy!

You can now click **"Proxy Hosts"** -> **"Add Proxy Host"** to seamlessly connect any domain name to any port running on your VPS, and click the SSL tab to generate a free certificate instantly. Say goodbye to manual Nginx configs!

---

## Conclusion

You now have Nginx Proxy Manager running on your VPS and fully accessible through its web dashboard. You can route any number of services to clean domain names and issue free Let's Encrypt SSL certificates in just a few clicks, with no manual Nginx configuration required.

Pair NPM with other self-hosted services running on the same VPS, such as WordPress or Nextcloud, to build a complete, production-ready self-hosted stack.

If you need a reliable server for your self-hosted stack, check out our [Premium VPS plans](/premium-vps/) or [Budget VPS options](/budget-vps/), which both feature fast NVMe SSD storage and a high-bandwidth network port.