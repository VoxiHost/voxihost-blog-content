---
image: /assets/images/blog/en/setup-portainer-ubuntu-debian/og-image.png
title: "Managing Docker via Browser – Installing Portainer on VPS (Ubuntu/Debian)"
description: "Learn how to install Portainer CE on an Ubuntu or Debian server to easily manage Docker containers from a graphical user interface (GUI) in your browser."
date: 2026-07-14
translationKey: "setup-portainer-ubuntu-debian"
locale: en
tags: ["docker", "portainer", "vps", "linux", "ubuntu", "debian", "containers"]
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - sl0ikkk
howto:
  name: "How to install Portainer on VPS"
  totalTime: "PT10M"
  yield: "A running Portainer dashboard ready for container management"
  tool:
    - "Ubuntu or Debian VPS"
    - "SSH Client"
  steps:
    - name: "Step 1"
      text: "Connect to server and install Docker"
      url: "step-1-connect-to-your-server-and-install-docker"
    - name: "Step 2"
      text: "Prepare storage"
      url: "step-2-prepare-storage-for-portainer"
    - name: "Step 3"
      text: "Run Portainer"
      url: "step-3-run-portainer"
    - name: "Step 4"
      text: "First browser login"
      url: "step-4-first-login-in-the-browser"
    - name: "Step 5"
      text: "Managing containers"
      url: "step-5-managing-containers"
---

## Introduction

Docker is an amazing tool, but typing commands into a black terminal can be overwhelming for beginners. Enter **Portainer** – a graphical user interface (GUI) dashboard that lets you click your way through Docker straight from your web browser.

In this step-by-step tutorial, we will show you how to install Portainer on your server in just a few minutes. You don't need to be a Linux expert — just copy and paste the commands below!

> **Prerequisites:** You will need a fresh [VPS](/premium-vps/) running Ubuntu or Debian (e.g., from <span class="text-white">Voxi</span><span class="text-amber-300">Host</span>) and an SSH client (like PuTTY or the built-in Terminal) to connect to your server.

---

## Step 1: Connect to your Server and Install Docker

First, connect to your server via SSH. Next, let's make sure Docker is installed. If you don't have it yet, simply paste this one-line installer command which will do all the heavy lifting for you:

{% image "/assets/images/blog/setup-portainer-ubuntu-debian/H1.png", "Terminal showing the Docker installation process", "(max-width: 768px) 100vw, 800px" %}

```bash
curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh
```

Wait a few moments for the installation to finish.

---

## Step 2: Prepare Storage for Portainer

Portainer needs a safe place on your hard drive to save your passwords and configurations. Paste the following command into your terminal and press ENTER to create a "volume":

{% image "/assets/images/blog/setup-portainer-ubuntu-debian/H2.png", "Creating a Docker volume in the terminal", "(max-width: 768px) 100vw, 800px" %}

```bash
docker volume create portainer_data
```

---

## Step 3: Run Portainer

Now we will start Portainer itself. The command below will download the latest version and run it in the background. Paste the entire command into the terminal and press ENTER:

{% image "/assets/images/blog/setup-portainer-ubuntu-debian/H3.png", "Running the Portainer container with required parameters", "(max-width: 768px) 100vw, 800px" %}

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

{% image "/assets/images/blog/setup-portainer-ubuntu-debian/H4.png", "Portainer initial admin setup screen in the browser", "(max-width: 768px) 100vw, 800px" %}

---

## Step 5: Managing Containers

After logging in, click the **"Get Started"** button, and on the next screen, click the whale icon labeled **"Local"** (which represents the server we are currently on).

You're all set! You now have access to the full dashboard. From now on, you don't need to use terminal commands – you can manage your apps using your mouse.

{% image "/assets/images/blog/setup-portainer-ubuntu-debian/H5.png", "Portainer main dashboard showing active containers", "(max-width: 768px) 100vw, 800px" %}

---

## Conclusion

Portainer is an incredible tool that makes Docker accessible even for those without much Linux experience. If you need a server to learn and experiment, check out our [Premium VPS plans](/premium-vps/)!