---
image: /assets/images/blog/en/setup-docker-compose/og-image.png
title: 'How to Set Up Docker Compose: A Complete Guide to Managing Multi-Container Apps'
description: A complete step-by-step guide to setting up and using Docker Compose V2 on Ubuntu, Debian, AlmaLinux, Rocky Linux, CentOS, and Fedora.
date: '2026-03-25'
updated: '2026-06-02'
translationKey: setup-docker-compose
category: Tutorials
tags:
  - docker compose
  - docker
  - containers
  - linux
  - vps
  - server administration
  - ubuntu
  - almalinux
  - yaml
howto:
  name: How to Set Up and Use Docker Compose
  totalTime: PT15M
  yield: A multi-container application running smoothly via a docker-compose.yml file
  tool:
    - A VPS or dedicated server running a Linux distribution
    - Docker Engine officially installed
    - A user account with permissions to run Docker commands
  steps:
    - name: Verify Docker Compose plugin is installed
      text: Run docker compose version to ensure V2 is actively running on your system.
      url: step-1-verify-docker-compose-is-installed
    - name: Create a project directory
      text: Use mkdir my-project and cd into it to keep your deployment organized.
      url: step-2-set-up-a-project-directory
    - name: Write your docker-compose.yml file
      text: Create the YAML configuration detailing your web service, database, and ports.
      url: step-3-create-the-docker-compose-yml-file
    - name: Deploy your stack
      text: Run docker compose up -d to spin up the containers in the background.
      url: step-4-spin-it-up
    - name: Manage your stack
      text: Use docker compose logs, pause, stop, or down to manage the running environments.
      url: step-5-manage-your-environment
faq:
  - question: "What is the difference between Docker Compose V1 and V2?"
    answer: "Docker Compose V1 was a standalone Python command written as <code>docker-compose</code>. Docker Compose V2 is written in Go and integrated directly into the Docker CLI as a native plugin, executed as <code>docker compose</code>."
  - question: "How does Docker Compose handle container networks?"
    answer: "By default, Docker Compose automatically creates a single private network for all services defined in the YAML file. Each container joins this network, allowing them to resolve and communicate with other services using their service name (e.g. <code>database</code>)."
  - question: "Where is Docker Compose data stored when a container is deleted?"
    answer: "If you define a persistent volume under the <code>volumes:</code> key in your YAML file, Docker stores that data in a directory managed by the Docker engine on the host system. This data survives container restarts and deletions."
  - question: "How do I secure ports exposed by Docker Compose?"
    answer: "By default, mapping a port like <code>8080:80</code> exposes it globally. To limit access, map it to the localhost loopback address: <code>127.0.0.1:8080:80</code>. This restricts external access unless routed through a local reverse proxy."
  - question: "What does the depends_on option do in docker-compose.yml?"
    answer: "The <code>depends_on</code> option defines startup order dependencies. It ensures that prerequisite services (like databases) are started before the dependent services (like web applications) are launched."
status: published
locale: en
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Docker Engine is fantastic for running single, isolated containers. But modern applications rarely exist in total isolation. You usually have a web server, a backend API, a database (like MySQL or PostgreSQL), and maybe a caching layer (like Redis). 

Trying to start all of those containers manually and manually linking their internal networking together via exhausting, mile-long `docker run` commands is frustrating and severely prone to human error.

Enter **Docker Compose**. It allows you to declare your entire application stack in a single, clean `.yml` (YAML) configuration file. With one central command, Docker builds the internal networks, pulls all necessary images, and launches the entire stack sequentially.

## Step 1: Verify Docker Compose is Installed

If you followed our Docker installation guides for [Ubuntu/Debian](/blog/install-docker-ubuntu-debian/) or [AlmaLinux/Rocky/Fedora](/blog/install-docker-centos-rhel/), you actually already have Docker Compose installed! 

Modern Docker distributions have shifted away from the old standalone `docker-compose` binary (written in Python) to a native **Docker Compose V2 Plugin** (written in Go) embedded directly into the Docker CLI.

Verify it is installed by checking its version:

{% image "/assets/images/blog/en/setup-docker-compose/H1.png", "Running docker compose version in the terminal on Ubuntu to verify Docker Compose V2 is installed", "(max-width: 768px) 100vw, 800px" %}

```bash
docker compose version
```
*(Notice there is a space between `docker` and `compose`, not a hyphen!)* 

You should see an output like:
`Docker Compose version v2.32.x`

If you receive a "command not found" error, you need to install the plugin via your package manager:
- **Ubuntu/Debian**: `sudo apt install docker-compose-plugin`
- **AlmaLinux/RHEL**: `sudo dnf install docker-compose-plugin`

## Step 2: Set Up a Project Directory

Docker Compose relies absolutely on the context of the directory you run the command in. It looks for a file named `docker-compose.yml` in whatever folder you are currently inside. 

First, let's make a dedicated home for your new project so files stay organized:

{% image "/assets/images/blog/en/setup-docker-compose/H2.png", "Running mkdir my-webapp and cd my-webapp to create and enter a dedicated Docker Compose project directory on Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
mkdir my-webapp
cd my-webapp
```

## Step 3: Create the docker-compose.yml File

Now, let's create a functional, real-world example. We are going to deploy the official WordPress image and attach it to a secure MySQL database backend, cleanly configuring everything through Compose.

Open a new file with `nano`:

{% image "/assets/images/blog/en/setup-docker-compose/H3.png", "Running nano docker-compose.yml to open the Docker Compose configuration file for editing on Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
nano docker-compose.yml
```

Paste the following YAML block entirely:

```yaml
services:
  database:
    image: mysql:8.0
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
      WORDPRESS_DB_HOST: database
      WORDPRESS_DB_USER: wp_user
      WORDPRESS_DB_PASSWORD: secure_wp_password
      WORDPRESS_DB_NAME: wordpress
    depends_on:
      - database

volumes:
  db_data:
```

### Breaking Down the Configuration:
- **services**: We defined two containers: `database` and `wordpress`.
- **image**: Tells Docker which container template to pull from Docker Hub.
- **environment**: Injects secure variables (like passwords and usernames) automatically into the containers so they configure themselves silently on first boot. Look at how the `wordpress` container knows its host is `database` (the exact name of the other service). Docker Compose automatically built an internal, invisible network for them to talk to each other seamlessly!
- **ports**: Maps port `8080` on the public internet directly to port `80` (Standard HTTP) inside the internal isolated WordPress container.
- **volumes**: In standard containers, when you delete a container, all the data goes with it. We mapped the database storage to a hard drive volume so your data persists even if you restart or delete the container!
- **depends_on**: Ensures WordPress does not attempt to boot up until the database is successfully running.

Save and exit.

## Step 4: Spin it Up

With one file, your entire infrastructure is declared. To launch it, run the `up` command. The `-d` flag tells it to run "detached" in the background so you can keep using your terminal console:

{% image "/assets/images/blog/en/setup-docker-compose/H4.png", "Running docker compose up -d to start all containers defined in docker-compose.yml in detached mode on Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
docker compose up -d
```

Docker will automatically:
1. Create a dedicated internal network for `my-webapp`.
2. Pull the heavy MySQL and WordPress images.
3. Start the database, assign the passwords, and build the persistent storage partition.
4. Start the WordPress server, attach it to the network, and map port 8080.

Once finished, open your web browser and navigate to your server's IP address combined with port 8080:
`http://your_server_ip:8080`

> **Security Warning:** Docker manages its own network rules through `iptables`. When you use a `ports:` mapping in your `docker-compose.yml` file, Docker will **bypass your UFW firewall completely**. To keep a service private, map it to `127.0.0.1:8080` instead of just `8080`.

{% image "/assets/images/blog/en/setup-docker-compose/H5.png", "WordPress installation wizard screen in browser after launching a WordPress plus MySQL Docker Compose stack on Ubuntu VPS", "(max-width: 768px) 100vw, 800px" %}

You will instantly hit the famous WordPress installation screen!

## Step 5: Manage Your Environment

Here are the crucial commands to memorize when operating in the directory containing your `docker-compose.yml` file:

See what's actively running in this project:

{% image "/assets/images/blog/en/setup-docker-compose/H6.png", "Running docker compose ps to list all running containers and their ports in the current Docker Compose project", "(max-width: 768px) 100vw, 800px" %}

```bash
docker compose ps
```

Check the deeply detailed background system logs (useful if an app crashes to see why it died):
```bash
docker compose logs
# Add -f to follow the logs live in real-time
docker compose logs -f
```

Stop the containers temporarily without deleting them:

{% image "/assets/images/blog/en/setup-docker-compose/H7.png", "Running docker compose stop to temporarily stop all running containers in a Docker Compose project without removing them", "(max-width: 768px) 100vw, 800px" %}

```bash
docker compose stop
```
*(You can start them again using `docker compose start`)*

Tear the entire project down (stops the containers, deletes them, and removes the internal network):

{% image "/assets/images/blog/en/setup-docker-compose/H8.png", "Running docker compose down to stop and remove all containers, networks, and volumes in a Docker Compose project", "(max-width: 768px) 100vw, 800px" %}

```bash
docker compose down
```
*(By default, this does NOT delete your database volume, so your WordPress posts are completely safe upon re-deployment).*

To deploy high-availability application stacks using containerized architecture, there is nothing like a high-performance backend infrastructure backing it. Spin up a [Premium VPS](/premium-vps/), install Docker, copy in your YAML configs, and launch your complex, multi-layered infrastructures into production effortlessly.