---
image: /assets/images/blog/en/install-docker-centos-rhel/og-image.png
title: 'How to Install Docker on AlmaLinux, CentOS, Rocky Linux & Fedora: The Complete Server Guide'
description: A complete step-by-step guide to installing the latest official Docker Engine and Docker Compose on AlmaLinux, CentOS Stream, Rocky Linux, and Fedora servers.
date: '2026-03-25'
translationKey: install-docker-almalinux-centos-rocky
category: Tutorials
tags:
  - docker
  - almalinux
  - centos
  - rocky linux
  - fedora
  - docker engine
  - containers
  - linux
  - vps
  - server administration
  - docker compose
howto:
  name: How to Install Docker Engine on AlmaLinux, CentOS Stream, Rocky Linux & Fedora
  totalTime: PT10M
  yield: A fully configured RHEL-family server running the latest Docker Engine and Docker Compose plugin
  tool:
    - A VPS or dedicated server running AlmaLinux, CentOS Stream, Rocky Linux, or Fedora
    - SSH client (e.g. terminal, PuTTY)
    - A user account with sudo privileges
  steps:
    - name: Remove old conflicting packages
      text: Run sudo dnf remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine to clear conflicts.
      url: step-1-remove-old-versions
    - name: Set up the Docker repository
      text: Install yum-utils and use yum-config-manager to add the official Docker repository.
      url: step-2-set-up-the-docker-repository
    - name: Install Docker Engine
      text: Run sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y.
      url: step-3-install-docker-engine
    - name: Start and enable the service
      text: Run sudo systemctl enable --now docker to start the daemon.
      url: step-4-start-and-enable-docker
    - name: Verify installation
      text: Run sudo docker run hello-world to confirm everything works.
      url: step-5-verify-the-installation
faq:
  - question: "Why should I remove old Docker versions before installation?"
    answer: "Older packages (like <code>docker</code> or <code>docker-engine</code>) conflict with the official <code>docker-ce</code> (Community Edition) package, which is maintained by Docker. Removing them prevents dependency resolution issues."
  - question: "What is docker-ce-cli and why is it installed separately?"
    answer: "The <code>docker-ce-cli</code> is the command-line interface tool used to interact with the Docker daemon. Keeping the CLI package separate allows you to manage remote Docker daemons from a local machine without installing the engine itself."
  - question: "How do I run Docker commands without using sudo?"
    answer: "You can add your user to the docker group by running <code>sudo usermod -aG docker $USER</code>. You will need to log out and log back in for this change to take effect."
  - question: "How do I check if the Docker daemon is currently running?"
    answer: "You can check the status of the Docker service by running the systemctl command: <code>sudo systemctl status docker</code>."
  - question: "What is the difference between Docker and podman on RHEL systems?"
    answer: "Podman is a daemonless container engine developed by Red Hat as an alternative to Docker. While podman is the default on standard RHEL installations, Docker is still widely preferred due to its mature ecosystem and native Docker Compose plugin support."
status: published
locale: en
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Docker revolutionized server deployments by enabling applications to be isolated into lightweight, portable, self-contained units called containers. No matter what your underlying operating system is, a Docker container spins up exactly the same way.

However, many default repositories (like those built into RHEL, AlmaLinux, CentOS, and Rocky Linux) often point to Podman instead of Docker, or they host severely outdated versions of the Docker Engine. To guarantee you have access to the latest security features and the integrated `docker-compose-plugin`, you need to pull it directly from Docker's official source.

## Step 1: Remove Old Versions

Before you install the official engine, you need to verify no older, conflicting packages are lingering on the system (even if you never installed them yourself). These packages usually orbit the names `docker` or `docker-engine`.

Run this command to clear the slate smoothly:

{% image "/assets/images/blog/en/install-docker-centos-rhel/H1.png", "Running sudo dnf remove docker to clean up any old conflicting Docker packages on AlmaLinux or Rocky Linux before a fresh install", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo dnf remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

It is perfectly fine if `dnf` reports that none of these packages are installed.

## Step 2: Set up the Docker Repository

You need to tell your package manager (`dnf`) exactly where to find the official Docker releases. Docker provides a convenient configuration tool natively supported by RHEL systems via the `yum-utils` package.

Install the utilities:

{% image "/assets/images/blog/en/install-docker-centos-rhel/H2.png", "Running sudo dnf install -y yum-utils on AlmaLinux to install the yum-config-manager tool needed to add the Docker CE repository", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo dnf install -y yum-utils
```

Now, use the `yum-config-manager` to safely add the official Docker repository to your system sources:

{% image "/assets/images/blog/en/install-docker-centos-rhel/H3.png", "Running sudo yum-config-manager --add-repo to add the official Docker CE repository to AlmaLinux or Rocky Linux", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
*(Even if you are on AlmaLinux, Rocky Linux, or Fedora, passing the `/centos/` path is correct, as they share the absolute exact same binary architecture for Enterprise Linux).*

## Step 3: Install Docker Engine

With the repository securely added, your system knows where to look. You can now install the entire Docker suite. 

This command installs the core engine (`docker-ce`), the command-line interface (`docker-ce-cli`), the container runtime (`containerd.io`), and modern plugins like **Docker Compose V2** (`docker-compose-plugin`).

{% image "/assets/images/blog/en/install-docker-centos-rhel/H4.png", "Running sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin on AlmaLinux to install Docker Engine and Compose", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

## Step 4: Start and Enable Docker

Unlike Ubuntu, which automatically starts services immediately after downloading them, RHEL-family distributions prefer you to intentionally start them. 

You need to start the Docker daemon and enable it so it safely wakes up every time the server reboots. You can do both in one single systemctl command:

{% image "/assets/images/blog/en/install-docker-centos-rhel/H5.png", "Running sudo systemctl enable --now docker on AlmaLinux to start the Docker service and enable it to launch automatically on boot", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl enable --now docker
```

To confirm the service is alive, check the status:

{% image "/assets/images/blog/en/install-docker-centos-rhel/H6.png", "Running sudo systemctl status docker on AlmaLinux to confirm the Docker daemon is active and running correctly", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl status docker
```
Look for the bright green `"active (running)"` text.

## Step 5: Verify the Installation

To prove unequivocally that Docker can successfully pull images from the internet and spin them up into operating containers, use the standard test payload:

{% image "/assets/images/blog/en/install-docker-centos-rhel/H7.png", "Running sudo docker run hello-world on AlmaLinux or Rocky Linux to verify Docker Engine is installed and working correctly", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo docker run hello-world
```

If your configuration is correct, a container will be downloaded, run its code, and output a large confirmation block beginning with:
> *"Hello from Docker! This message shows that your installation appears to be working correctly."*

> **Docker and your Firewall:** Docker manages its own network rules directly through `iptables`. While RedHat systems use `firewalld`, which is generally more integrated with Docker than UFW, it's still best practice to be careful when exposing ports via the `-p` or `--publish` flag. Always verify your open ports with `sudo firewall-cmd --list-all`.

## Step 6 (Optional): Run Docker Without Sudo

You probably noticed you had to type `sudo` to run the test script. By default, the Docker daemon binds to a Unix socket instead of a TCP port, and that socket is owned by the `root` user. 

If you created your own user account (as outlined in our [User Management Guide](/blog/add-sudo-user-centos/)), typing `sudo` hundreds of times a day can be tedious. You can grant your user native Docker rights by adding them to the `docker` group.

```bash
sudo usermod -aG docker $USER
```

**Warning:** The `docker` group grants privileges that are functionally equivalent to root access. Only add heavily trusted users to this group.

To force the system to acknowledge your new group standing without having to log out entirely:

```bash
su - $USER
```

Now, re-run the test without the sudo prefix:
```bash
docker run hello-world
```

If it works, congratulations! Your **VPS** is fully equipped to deploy the infinite number of pre-packaged container apps available on Docker Hub natively and securely.

For a reliable environment supporting rapid project prototyping with Docker, checkout our high-performance line of affordable [Budget VPS](/budget-vps/) environments today.