---
image: /assets/images/blog/en/ssl-letsencrypt-certbot-ubuntu/og-image.png
title: 'How to Set Up SSL with Let''s Encrypt & Certbot on Ubuntu & Debian: The Complete Guide'
description: A complete beginner-friendly guide to securing your Nginx or Apache web server with free SSL/TLS certificates from Let's Encrypt using Certbot on Ubuntu and Debian.
date: '2026-03-25'
updated: '2026-06-02'
translationKey: setup-ssl-letsencrypt-certbot
category: Tutorials
tags:
  - ssl
  - lets encrypt
  - certbot
  - https
  - ubuntu
  - debian
  - nginx
  - apache
  - security
  - linux
  - vps
howto:
  name: How to Set Up SSL with Let's Encrypt and Certbot on Ubuntu/Debian
  totalTime: PT10M
  yield: A fully secured web server with an automatically renewing SSL certificate and forced HTTPS connections
  tool:
    - A VPS or dedicated server running Ubuntu or Debian
    - An installed web server (Nginx or Apache) with a configured Virtual Host/Server Block
    - A registered domain name pointing to your server's public IP address
    - A user account with sudo privileges
  steps:
    - name: Install Certbot
      text: Run sudo apt install certbot and either python3-certbot-nginx or python3-certbot-apache.
      url: step-1-install-certbot
    - name: Allow HTTPS through UFW Firewall
      text: Ensure your firewall allows HTTPS traffic (port 443) using ufw allow 'Nginx Full' or 'Apache Full'.
      url: step-2-confirm-firewall-settings
    - name: Obtain the SSL Certificate
      text: Run sudo certbot --nginx -d your_domain.com or sudo certbot --apache -d your_domain.com.
      url: step-3-obtain-and-install-the-ssl-certificate
    - name: Verify Auto-Renewal
      text: Check if the automatic renewal timer is active by running sudo systemctl status certbot.timer.
      url: step-4-verify-auto-renewal
faq:
  - question: "What is the difference between SSL and TLS?"
    answer: "TLS (Transport Layer Security) is the modern, more secure successor to SSL (Secure Sockets Layer). Although the terms are often used interchangeably and most people still refer to them as SSL, modern secure web traffic actually uses the TLS protocol."
  - question: "Why do Let's Encrypt certificates only last for 90 days?"
    answer: "Short-lived certificates encourage automation, making manual renewal obsolete. They also limit the damage from compromised keys and ensure that abandoned domains automatically lose their active certificates quickly."
  - question: "How does Certbot prove that I own the domain?"
    answer: "Certbot uses the ACME protocol to complete a challenge. For Nginx and Apache, it temporary modifies your web server configuration or places a validation file in your web root to prove to Let's Encrypt that the domain resolves to your server."
  - question: "How do I force all traffic to use HTTPS instead of HTTP?"
    answer: "During installation, Certbot will ask if you want to redirect HTTP traffic. If you agree, it automatically adds a redirect rule (e.g. <code>return 301 https://$host$request_uri;</code> for Nginx) directly into your configuration files."
  - question: "Can I get a wildcard certificate using Let's Encrypt and Certbot?"
    answer: "Yes, but wildcard certificates (e.g., <code>*.your_domain.com</code>) require a DNS-01 challenge instead of HTTP. You must use a Certbot DNS plugin (like Cloudflare, Route53, or DigitalOcean) to automate adding TXT records to your DNS zone."
status: published
locale: en
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

In the modern web, serving your website over plain HTTP is no longer acceptable. Browsers will aggressively warn users that your site is "Not Secure," search engines like Google will heavily penalize your SEO rankings, and any submitted forms (like passwords or credit cards) will be transmitted in plain text for anyone to intercept.

You need an SSL/TLS certificate to enable HTTPS. Years ago, this was an expensive and deeply frustrating process. Today, thanks to the non-profit **Let's Encrypt** project and their automated client called **Certbot**, you can get enterprise-grade, cryptographically secure certificates completely for free in about two minutes.

## Prerequisites

Before you begin, you must have two things in place:
1. **A web server with a configured domain block**: You must have either Nginx or Apache installed, and a Server Block (Nginx) or Virtual Host (Apache) officially configured for your domain name. If you haven't done this, check out our [Nginx installation guide](/blog/install-nginx-ubuntu-debian/) or [Apache installation guide](/blog/install-apache-ubuntu-debian/).
2. **Proper DNS settings**: Your domain (e.g., `your_domain.com` and `www.your_domain.com`) must have `A` records actively pointing to your server's public IP address. Certbot will fail if the domain doesn't resolve to your server.

## Step 1: Install Certbot

Certbot is the tool that reaches out to the Let's Encrypt servers, proves you own the domain, downloads the certificates, and injects them directly into your web server's configuration files.

First, update your package index:
```bash
sudo apt update
```

Next, you need to install Certbot along with its plugin for your specific web server. 

**If you are using Nginx:**

{% image "/assets/images/blog/en/ssl-letsencrypt-certbot-ubuntu/H1.png", "Running sudo apt install certbot python3-certbot-nginx -y on Ubuntu/Debian - terminal output", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt install certbot python3-certbot-nginx -y
```

**If you are using Apache:**
```bash
sudo apt install certbot python3-certbot-apache -y
```

## Step 2: Confirm Firewall Settings

Certbot needs to communicate over HTTP (Port 80) to validate your domain, and your secure website will be served over HTTPS (Port 443). 

If you followed our [UFW Firewall guide](/blog/configure-ufw-ubuntu-debian/), you need to ensure the firewall is allowing this traffic. 

**For Nginx:**
```bash
sudo ufw status
```
If you only see `Nginx HTTP` allowed, you need to upgrade to `Nginx Full`:
```bash
sudo ufw allow 'Nginx Full'
sudo ufw delete allow 'Nginx HTTP'
```

**For Apache:**
If you only see `Apache` allowed in your `sudo ufw status`, upgrade the profile:
```bash
sudo ufw allow 'Apache Full'
sudo ufw delete allow 'Apache'
```

## Step 3: Obtain and Install the SSL Certificate

This is where the magic happens. By using the web server plugins you installed in Step 1, Certbot will handle the entire validation, downloading, and configuration process automatically.

**Run Certbot for Nginx:**

{% image "/assets/images/blog/en/ssl-letsencrypt-certbot-ubuntu/H2.png", "Running sudo certbot --nginx -d your_domain.com -d www.your_domain.com on Ubuntu/Debian - terminal output", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo certbot --nginx -d your_domain.com -d www.your_domain.com
```

**Run Certbot for Apache:**
```bash
sudo certbot --apache -d your_domain.com -d www.your_domain.com
```

### The Certbot Prompts

When you run the command for the first time, Certbot will ask you a series of questions:

1. **Email Address**: You must provide a valid email address. Let's Encrypt uses this strictly to notify you of impending expirations (if auto-renewal fails) or major security events.
2. **Terms of Service**: Type `Y` to agree to the Let's Encrypt TOS.
3. **EFF Mailing List**: Type `Y` or `N` based on whether you want promotional emails from the Electronic Frontier Foundation (the creators of Certbot).

Certbot will then communicate with the Let's Encrypt API and run a challenge to verify you actually control the domain. 

If it succeeds, it will automatically edit your Nginx `.conf` or Apache Virtual Host file to enable HTTPS. Modern versions of Certbot will automatically configure your server to aggressively redirect all unencrypted HTTP traffic entirely to the secure HTTPS connection.

When it finishes, navigate to `https://your_domain.com` in your browser to verify the padlock icon appears!

## Step 4: Verify Auto-Renewal

Let's Encrypt certificates are absolutely free, but to minimize the impact of stolen or abandoned certificates, they only last for **90 days**.

Thankfully, you never have to repeat Step 3. The `certbot` package on Ubuntu and Debian installs a systemd timer (a background background scheduled task) that runs twice a day. It checks for any certificates expiring in the next 30 days and smoothly auto-renews them in the background without dropping web traffic.

You can verify the timer is active by running:

{% image "/assets/images/blog/en/ssl-letsencrypt-certbot-ubuntu/H3.png", "Running sudo systemctl status certbot.timer on Ubuntu/Debian - terminal output", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl status certbot.timer
```

You should see `"Active: active (waiting)"`. 

To test the renewal process and ensure there are no configuration errors blocking it, you can run a dry run:
```bash
sudo certbot renew --dry-run
```

If the dry run finishes without any errors, you are successfully set up! Your server is now continuously and permanently secured.

Need a blistering-fast environment ready to deploy secure web apps? Pick up an exceptionally affordable [Budget VPS](/budget-vps/), setup your Nginx block, slap a free SSL on it, and launch your project safely to the world.