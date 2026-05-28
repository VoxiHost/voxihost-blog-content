---
image: /assets/images/blog/install-docker-ubuntu-debian/og-image.png
title: 'How to Install Docker on Ubuntu & Debian: The Complete Server Guide'
description: A complete step-by-step guide to installing the latest Docker Engine and Docker Compose on Ubuntu and Debian servers properly using the official Docker repository.
date: '2026-03-25'
translationKey: install-docker-ubuntu-debian
category: Tutorials
tags:
  - docker
  - ubuntu
  - debian
  - docker engine
  - containers
  - linux
  - vps
  - server administration
  - docker compose
howto:
  name: How to Install Docker Engine on Ubuntu and Debian
  totalTime: PT10M
  yield: A server running the latest official release of Docker Engine and Docker Compose
  tool:
    - A VPS or dedicated server running Ubuntu or Debian
    - SSH client (e.g. terminal, PuTTY)
    - A user account with sudo privileges
  steps:
    - name: Update the server and install prerequisite packages
      text: Run sudo apt update and install curl, ca-certificates, and gnupg.
      url: step-1-update-system-and-install-dependencies
    - name: Add Docker's official GPG key
      text: Download Docker's encryption key and save it to /etc/apt/keyrings to verify the packages.
      url: step-2-add-dockers-official-gpg-key
    - name: Set up the Docker repository
      text: Add the official repository to your apt sources list and run apt update.
      url: step-3-set-up-the-docker-repository
    - name: Install Docker Engine
      text: Run sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin.
      url: step-4-install-docker-engine-and-compose
    - name: Verify installation
      text: Run sudo docker run hello-world to verify the engine is successfully running containers.
      url: step-5-verify-the-installation
status: published
locale: en
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Docker has fundamentally changed how applications are developed and deployed. Instead of wrestling with dependencies ("it works on my machine but not the server!"), Docker packages your application and everything it needs into an isolated, portable **container** that runs perfectly anywhere Docker is installed.

While Ubuntu and Debian include a version of Docker in their default `apt` repositories, it is almost always outdated. To get the latest features, security patches, and the integrated `docker-compose-plugin`, you need to install Docker Engine directly from Docker's official repository.

## Step 1: Update System and Install Dependencies

Before adding a new external repository, update your package index and ensure you have the necessary base packages required over HTTPS. 

{% image "/assets/images/blog/install-docker-ubuntu-debian/H1.png", "Running sudo apt update and installing ca-certificates curl gnupg prerequisites for Docker on Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt update
sudo apt install ca-certificates curl gnupg -y
```

## Step 2: Add Docker's Official GPG Key

To ensure the software you are downloading hasn't been tampered with, `apt` uses GPG keys. You need to download Docker's official GPG key and store it properly in the new standardized `/etc/apt/keyrings` directory.

Create the directory (it may already exist):

{% image "/assets/images/blog/install-docker-ubuntu-debian/H2.png", "Running sudo install -m 0755 -d /etc/apt/keyrings to create the GPG keyrings directory for Docker on Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

Download and save the key:

{% image "/assets/images/blog/install-docker-ubuntu-debian/H3.png", "Downloading and adding Docker's official GPG key to /etc/apt/keyrings on Ubuntu using curl and gpg", "(max-width: 768px) 100vw, 800px" %}

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
*(If you are on Debian, change `ubuntu` in the URL to `debian`)*

Finally, adjust the permissions so `apt` can read the key during updates:

{% image "/assets/images/blog/install-docker-ubuntu-debian/H4.png", "Running sudo chmod a+r to set read permissions on the Docker GPG keyring file on Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

## Step 3: Set up the Docker Repository

Now that the system trusts incoming packages from Docker, you need to tell `apt` where to find them by adding the repository to your sources list.

Copy and paste this entire block of code into your terminal and press Enter. It automatically detects your system's architecture (like `amd64` or `arm64`) and code name, and creates the file for you:

{% image "/assets/images/blog/install-docker-ubuntu-debian/H5.png", "Adding the official Docker CE apt repository to /etc/apt/sources.list.d on Ubuntu or Debian", "(max-width: 768px) 100vw, 800px" %}

*(For Ubuntu)*

```bash
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

*(For Debian)*
```bash
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

With the new repository added, update your package index one more time so `apt` sees the new Docker packages:

```bash
sudo apt update
```

## Step 4: Install Docker Engine and Compose

Finally, you can install the latest stable version of Docker cleanly. 

This command installs the core Docker engine (`docker-ce`), the command-line client (`docker-ce-cli`), the runtime (`containerd.io`), and crucial plugins like **Docker Compose V2** (`docker-compose-plugin`).

{% image "/assets/images/blog/install-docker-ubuntu-debian/H6.png", "Running sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin on Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

The Docker daemon automatically starts running as soon as the installation finishes.

## Step 5: Verify the Installation

To confirm that the entire engine is working, you can download and run Docker's official test image. It spins up a tiny container, prints a confirmation message to your terminal, and then shuts itself down.

{% image "/assets/images/blog/install-docker-ubuntu-debian/H7.png", "Running sudo docker run hello-world on Ubuntu to verify Docker Engine installation is working", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo docker run hello-world
```

If it succeeds, you'll see a lengthy validation message starting with:
> *"Hello from Docker! This message shows that your installation appears to be working correctly."*

> **Docker and your Firewall (UFW):** By default, Docker modifies your system's `iptables` rules directly. This means **Docker completely bypasses UFW rules**. If you launch a container and map a port to the public internet (e.g., `-p 8080:80`), that port will be open globally even if UFW is set to "deny". Always be cautious when exposing ports in production.

## Step 6 (Optional): Run Docker Without Sudo

By default, the `docker` command can only be run by the root user or someone using `sudo`. This can be annoying if you are managing a lot of containers.

If you created your own non-root sudo-user (as recommended in our [user guide](/blog/add-sudo-user-ubuntu/)), you can add that user to the `docker` group to bypass the `sudo` requirement.

```bash
sudo usermod -aG docker $USER
```

**Warning:** Being in the `docker` group effectively grants root-level privileges to the system. Only add users you completely trust.

For this group change to take effect, you must log out of your SSH session entirely and log back in, or run the following command to activate the group logic in your current shell:
```bash
su - $USER
```

Now, try running the test without `sudo`:
```bash
docker run hello-world
```

If it works, congratulations! Your **VPS** is fully equipped to deploy the infinite number of pre-packaged container apps available on Docker Hub natively and securely.

For a reliable environment supporting rapid project prototyping with Docker, checkout our high-performance line of affordable [Budget VPS](/budget-vps/) environments today.