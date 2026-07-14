---
image: /assets/images/blog/en/setup-portainer-ubuntu-debian/og-image.png
title: "How to Install Portainer on Ubuntu & Debian VPS"
description: "Learn how to install Portainer CE on an Ubuntu or Debian VPS to easily manage Docker containers from a graphical user interface (GUI) in your browser."
date: '2026-07-14'
translationKey: setup-portainer-ubuntu-debian
locale: en
category: Tutorials
tags:
  - docker
  - portainer
  - vps
  - linux
  - ubuntu
  - debian
  - containers
status: published
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - sl0ikkk
howto:
  name: How to Install Portainer CE on Ubuntu and Debian
  totalTime: PT10M
  yield: A running Portainer CE dashboard ready for Docker container management via browser
  tool:
    - A VPS or dedicated server running Ubuntu or Debian
    - SSH client (e.g. terminal, PuTTY)
    - A user account with sudo privileges
  steps:
    - name: Connect to your server and install Docker
      text: Connect via SSH and run the official Docker one-line installer script.
      url: step-1-connect-to-your-server-and-install-docker
    - name: Prepare storage for Portainer
      text: Create a persistent Docker volume to store Portainer's configuration and data.
      url: step-2-prepare-storage-for-portainer
    - name: Run Portainer
      text: Pull and start the Portainer CE container with the required port and volume mappings.
      url: step-3-run-portainer
    - name: First login in the browser
      text: Open the Portainer web UI and create your admin account.
      url: step-4-first-login-in-the-browser
    - name: Managing containers
      text: Navigate the Portainer dashboard and manage Docker containers without the terminal.
      url: step-5-managing-containers
faq:
  - question: "What is the difference between Portainer CE and Portainer Business Edition?"
    answer: "Portainer CE (Community Edition) is free and open-source - it covers all core Docker management features. Portainer Business Edition (BE) adds enterprise features such as role-based access control (RBAC), OAuth/LDAP integration, and extended support."
  - question: "Is it safe to expose port 9443 to the internet?"
    answer: "Portainer uses HTTPS on port 9443, but exposing it publicly is not recommended without additional protection. Consider placing it behind a reverse proxy (e.g. <code>Nginx Proxy Manager</code>) with authentication, or restricting access to trusted IP addresses via your firewall."
  - question: "How do I update Portainer to a newer version?"
    answer: "Stop and remove the existing container, then pull the latest image and run the same <code>docker run</code> command again. Your data is preserved in the <code>portainer_data</code> volume: <code>docker stop portainer && docker rm portainer</code>, then re-run the original install command."
  - question: "Does Portainer support Docker Compose?"
    answer: "Yes. Portainer calls them <strong>Stacks</strong>. You can deploy and manage Docker Compose files directly from the Portainer web UI without touching the terminal."
  - question: "What should I do if I forgot my Portainer admin password?"
    answer: "You need to reset the password via the command line. Stop the container, run <code>docker run --rm -v portainer_data:/data portainer/helper-reset-password</code>, copy the generated password, then restart Portainer with <code>docker start portainer</code>."
---

## Introduction

Docker is an amazing tool, but typing commands into a black terminal can be overwhelming for beginners. Enter **Portainer** - a graphical user interface (GUI) dashboard that lets you click your way through Docker straight from your web browser.

In this step-by-step tutorial, we will show you how to install Portainer on your server in just a few minutes. You don't need to be a Linux expert - just copy and paste the commands below!

> **Prerequisites:** You will need a [Premium VPS](/premium-vps/) running Ubuntu or Debian and an SSH client (e.g. PuTTY or your system Terminal). If you don't have a server yet, <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> offers [Premium VPS](/premium-vps/) and [Budget VPS](/budget-vps/) with Ubuntu and Debian available out of the box.

---

## Step 1: Connect to your Server and Install Docker

First, connect to your server via SSH. Next, let's make sure Docker is installed. If you don't have it yet, simply paste this one-line installer command which will do all the heavy lifting for you:

{% image "/assets/images/blog/en/setup-portainer-ubuntu-debian/H1.png", "Terminal showing the Docker installation process on Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh
```

Wait a few moments for the installation to finish.

---

## Step 2: Prepare Storage for Portainer

Portainer needs a safe place on your hard drive to save your passwords and configurations. Paste the following command into your terminal and press ENTER to create a "volume":

{% image "/assets/images/blog/en/setup-portainer-ubuntu-debian/H2.png", "Creating a Docker volume for Portainer in the terminal", "(max-width: 768px) 100vw, 800px" %}

```bash
docker volume create portainer_data
```

---

## Step 3: Run Portainer

Now we will start Portainer itself. The command below will download the latest version and run it in the background. Paste the entire command into the terminal and press ENTER:

{% image "/assets/images/blog/en/setup-portainer-ubuntu-debian/H3.png", "Running the Portainer CE container with required port and volume parameters", "(max-width: 768px) 100vw, 800px" %}

```bash
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

That's it! The installation on the server side is complete.

---

## Step 4: First Login in the Browser

You can now close your terminal. Open your favorite web browser (like Chrome or Firefox) and type the following into the address bar:

`https://YOUR_SERVER_IP:9443`
*(Replace "YOUR_SERVER_IP" with the actual IP address of your VPS, e.g., https://192.168.1.50:9443)*

> **Tip:** If you see the error `Client sent an HTTP request to an HTTPS server.`, it means your browser tried to connect via standard HTTP. Make sure your address explicitly starts with **https://**.

> **Important:** Your browser might show a "Your connection is not private" warning. This is completely normal because the server uses a self-signed certificate by default. Simply click **"Advanced"** and then **"Proceed / Continue"**.

You will see the initial setup screen. Create a strong password for yourself, type it twice, and click the **Create user** button.

> **Important (5-minute limit!):** Portainer takes security seriously. If more than 5 minutes pass between starting the container (Step 3) and setting your password, the setup will time out. To unlock it, go to your terminal and type `docker restart portainer`, then refresh the page. You will then be asked for a **Setup Token**. To get it, type `docker logs portainer`. Near the bottom of the logs, you will see *Use the following token to setup the administrator user:* followed by a long code. Copy this token and paste it into the browser.

{% image "/assets/images/blog/en/setup-portainer-ubuntu-debian/H4.png", "Portainer initial admin account setup screen in the browser", "(max-width: 768px) 100vw, 800px" %}

---

## Step 5: Managing Containers

After logging in, click the **"Get Started"** button, and on the next screen, click the whale icon labeled **"Local"** (which represents the server we are currently on).

You're all set! You now have access to the full dashboard. From now on, you don't need to use terminal commands - you can manage your apps using your mouse.

{% image "/assets/images/blog/en/setup-portainer-ubuntu-debian/H5.png", "Portainer main dashboard showing a list of active Docker containers", "(max-width: 768px) 100vw, 800px" %}

---

## Conclusion

Portainer is an incredible tool that makes Docker accessible even for those without much Linux experience. If you are looking for a server to run your own projects, <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> offers [Premium VPS](/premium-vps/) and [Budget VPS](/budget-vps/) with Ubuntu and Debian ready to go.