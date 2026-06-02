---
image: /assets/images/blog/en/install-nginx-ubuntu-debian/og-image.png
title: 'How to Install Nginx on Ubuntu & Debian: The Complete Server Guide'
description: A complete step-by-step guide to installing Nginx on Ubuntu and Debian servers. Learn how to install the web server, configure UFW, manage the Nginx process, and set up your first server block.
date: '2026-03-25'
updated: '2026-06-02'
translationKey: install-nginx-ubuntu-debian
category: Tutorials
tags:
  - nginx
  - ubuntu
  - debian
  - web server
  - linux
  - vps
  - server administration
  - server block
  - ufw
howto:
  name: How to Install Nginx on Ubuntu and Debian
  totalTime: PT15M
  yield: A fully functioning Nginx web server running on Ubuntu or Debian, ready to host websites
  tool:
    - A VPS or dedicated server running Ubuntu or Debian
    - SSH client (e.g. terminal, PuTTY)
    - A user account with sudo privileges
  steps:
    - name: Update the package index and install Nginx
      text: Run sudo apt update followed by sudo apt install nginx to install the web server.
      url: step-1-install-nginx
    - name: Configure the firewall
      text: Run sudo ufw allow 'Nginx Full' to open HTTP (80) and HTTPS (443) traffic.
      url: step-2-adjust-the-firewall
    - name: Verify Nginx is running
      text: Check your server's IP in a browser to see the default Nginx welcome page.
      url: step-3-check-your-web-server
    - name: Manage the Nginx process
      text: Learn to use systemctl to start, stop, restart, and reload Nginx.
      url: step-4-manage-the-nginx-process
    - name: Set up a Server Block (Virtual Host)
      text: Create a custom configuration file in /etc/nginx/sites-available/ to host your own domain.
      url: step-5-set-up-a-server-block-virtual-host
faq:
  - question: "What is Nginx and why is it preferred over Apache?"
    answer: "Nginx uses an asynchronous, event-driven architecture that handles concurrent connections much more efficiently than Apache's process-driven model, making it ideal for high-traffic sites and reverse proxy setups."
  - question: "What is the difference between sites-available and sites-enabled directories?"
    answer: "The <code>sites-available</code> directory contains all server block configuration files you have created, while <code>sites-enabled</code> contains symbolic links to the configurations you want Nginx to actually load and activate."
  - question: "How do I check Nginx configuration files for syntax errors?"
    answer: "You should always run the test command <code>sudo nginx -t</code> before reloading Nginx to ensure there are no syntax errors that could crash your web server."
  - question: "How do I redirect all HTTP traffic to HTTPS in Nginx?"
    answer: "In your HTTP server block (listening on port 80), add a directive: <code>return 301 https://$host$request_uri;</code> to redirect all incoming traffic to the secure HTTPS protocol."
  - question: "Where are the default Nginx error and access log files located?"
    answer: "By default, Nginx logs all web traffic and errors in the <code>/var/log/nginx/</code> directory, specifically in the <code>access.log</code> and <code>error.log</code> files."
status: published
locale: en
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Nginx is one of the most popular web servers in the world. Originally written to solve the "C10k problem" (handling 10,000 concurrent connections), it has grown into the default choice for high-performance websites. Whether you are serving static HTML files, running a Node.js API, or setting up a reverse proxy for a complex backend, Nginx is exactly what you need.

Installing Nginx on any Linux server running Ubuntu or Debian takes less than 15 minutes. 

## Step 1: Install Nginx

Because Nginx is available in the default Ubuntu and Debian repositories, it is straightforward to install using the `apt` package manager.

Before installing any new software, refresh your local package index:

```bash
sudo apt update
```

Now, install Nginx:

{% image "/assets/images/blog/en/install-nginx-ubuntu-debian/H1.png", "Running sudo apt install nginx -y on Ubuntu to install the Nginx web server from apt", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt install nginx -y
```

Ubuntu and Debian automatically start the Nginx service as soon as the installation finishes.

## Step 2: Adjust the Firewall

If you have enabled the UFW firewall (as we highly recommend in our [UFW Setup Guide](/blog/configure-ufw-ubuntu-debian/)), you need to allow connections to Nginx so the outside world can reach your websites.

During installation, Nginx registers itself with UFW to provide a few automatic application profiles. You can see them by typing:

{% image "/assets/images/blog/en/install-nginx-ubuntu-debian/H2.png", "Running sudo ufw app list on Ubuntu to view available Nginx firewall application profiles", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo ufw app list
```

You should see:
```text
Available applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH
```

- **Nginx HTTP**: Opens port 80 (normal, unencrypted web traffic).
- **Nginx HTTPS**: Opens port 443 (TLS/SSL encrypted traffic).
- **Nginx Full**: Opens both port 80 and 443.

For the vast majority of cases, you'll eventually want an SSL certificate, so it's best to allow both upfront:

{% image "/assets/images/blog/en/install-nginx-ubuntu-debian/H3.png", "Running sudo ufw allow 'Nginx Full' to open HTTP port 80 and HTTPS port 443 through UFW on Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo ufw allow 'Nginx Full'
```

Verify that the change was successfully applied:

```bash
sudo ufw status
```

## Step 3: Check your Web Server

At this point, your web server is fully operational. To verify this, open your favorite web browser and navigate to your server's public IP address. 

If you don't know your server's public IP address, you can find it by running:

{% image "/assets/images/blog/en/install-nginx-ubuntu-debian/H4.png", "Running curl -4 icanhazip.com on Ubuntu to retrieve the server's public IP address", "(max-width: 768px) 100vw, 800px" %}

```bash
curl -4 icanhazip.com
```

Type that IP address into your browser: `http://your_server_ip`

{% image "/assets/images/blog/en/install-nginx-ubuntu-debian/H5.png", "Default Nginx welcome page in browser confirming successful installation on Ubuntu server", "(max-width: 768px) 100vw, 800px" %}

You should see the default **"Welcome to nginx!"** landing page. This confirms that the software is running correctly and that your firewall is allowing traffic.

## Step 4: Manage the Nginx Process

Now that you have your web server up and running, you need to know how to manage it. You will use `systemctl` commands to control the Nginx service.

To stop your web server:

{% image "/assets/images/blog/en/install-nginx-ubuntu-debian/H6.png", "Running sudo systemctl stop nginx to stop the Nginx web server process on Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl stop nginx
```

To start the web server when it is stopped:

{% image "/assets/images/blog/en/install-nginx-ubuntu-debian/H7.png", "Running sudo systemctl start nginx to start the Nginx web server on Ubuntu or Debian", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl start nginx
```

To stop and then completely restart the service:

{% image "/assets/images/blog/en/install-nginx-ubuntu-debian/H8.png", "Running sudo systemctl restart nginx to fully restart Nginx and apply new configuration on Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl restart nginx
```

If you only made configuration changes (like adding a new domain), Nginx can reload without dropping active connections. This is the command you will use most often:

{% image "/assets/images/blog/en/install-nginx-ubuntu-debian/H9.png", "Running sudo systemctl reload nginx to reload Nginx config without interrupting active connections", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl reload nginx
```

## Step 5: Set Up a Server Block (Virtual Host)

By default, Nginx on Ubuntu has one server block enabled that serves documents out of a directory at `/var/www/html`. While this works well for a single site, it becomes unmanageable if you are hosting multiple sites. 

Instead of modifying the default setup, you should create a **Server Block** for your domain (in the Apache world, these are called Virtual Hosts).

For this example, we will use `your_domain.com`.

### 1. Create the directory for your domain

{% image "/assets/images/blog/en/install-nginx-ubuntu-debian/H10.png", "Running sudo mkdir -p to create a new website document root directory in /var/www on Ubuntu with Nginx", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo mkdir -p /var/www/your_domain.com/html
```

### 2. Assign ownership of the directory
Assign ownership to your current non-root user (so you can easily upload files later):
```bash
sudo chown -R $USER:$USER /var/www/your_domain.com/html
```

### 3. Create a sample index.html page

{% image "/assets/images/blog/en/install-nginx-ubuntu-debian/H11.png", "Creating a sample index.html test page with nano editor inside /var/www for Nginx on Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
nano /var/www/your_domain.com/html/index.html
```

Paste this simple HTML into the file:
```html
<html>
    <head>
        <title>Welcome to your_domain.com!</title>
    </head>
    <body>
        <h1>Success! The your_domain.com server block is working!</h1>
    </body>
</html>
```
Save and close the file.

### 4. Create the Nginx server block configuration
Now we need to tell Nginx to serve that folder when someone visits `your_domain.com`. Create a new configuration file in the `sites-available` directory:

{% image "/assets/images/blog/en/install-nginx-ubuntu-debian/H12.png", "Creating a new Nginx server block configuration file for a custom domain in sites-available on Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo nano /etc/nginx/sites-available/your_domain.com
```

Paste in the following configuration base:
```nginx
server {
    listen 80;
    listen [::]:80;

    root /var/www/your_domain.com/html;
    index index.html index.htm index.nginx-debian.html;

    server_name your_domain.com www.your_domain.com;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
Save and close the file.

### 5. Enable the file by linking it to sites-enabled
Nginx ignores files in `sites-available` unless they have a symbolic link placed into the `sites-enabled` directory. Create that link now:

{% image "/assets/images/blog/en/install-nginx-ubuntu-debian/H13.png", "Running sudo ln -s to create a symlink from sites-available to sites-enabled to activate Nginx virtual host", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo ln -s /etc/nginx/sites-available/your_domain.com /etc/nginx/sites-enabled/
```

### 6. Test your configuration and reload
Before restarting Nginx, always test your configuration to make sure there are no syntax errors:

{% image "/assets/images/blog/en/install-nginx-ubuntu-debian/H14.png", "Running sudo nginx -t to test Nginx configuration syntax before reloading the web server on Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo nginx -t
```

If the output says `syntax is ok` and `test is successful`, reload Nginx to apply the changes:

```bash
sudo systemctl reload nginx
```

Nginx should now be serving your domain name. Assuming you have updated your domain's DNS A-records to point to your VPS IP address, navigating to `http://your_domain.com` will show the success page you just created.

If you are looking for a highly reliable and blazing-fast environment to host your new Nginx server, spin up one of our [Premium VPS](/premium-vps/) instances and deploy your projects today.