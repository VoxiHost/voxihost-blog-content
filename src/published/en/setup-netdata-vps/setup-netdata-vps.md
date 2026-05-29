---
image: /assets/images/blog/en/setup-netdata-vps/og-image.png
title: How to Set Up Netdata for Real-Time VPS Monitoring
description: A complete step-by-step guide to installing Netdata on your Linux VPS. Get highly detailed, beautiful real-time dashboard metrics for CPU, RAM, Network, and Disk in minutes.
date: '2026-03-25'
translationKey: setup-netdata-vps
category: Tutorials
tags:
  - netdata
  - monitoring
  - linux
  - vps
  - server administration
  - dashboard
  - metrics
howto:
  name: How to Set Up Netdata for Server Monitoring
  totalTime: PT10M
  yield: A comprehensive, beautifully graphed real-time monitoring dashboard accessible via a web browser
  tool:
    - A VPS or dedicated server running any mainstream Linux distribution
    - SSH client (e.g. terminal, PuTTY)
    - A user account with sudo privileges
  steps:
    - name: Install Netdata
      text: 'Run the official one-line kickstart script: wget -O /tmp/netdata-kickstart.sh https://get.netdata.cloud/kickstart.sh && sh /tmp/netdata-kickstart.sh.'
      url: step-1-install-netdata-using-the-kickstart-script
    - name: Configure the Firewall
      text: Allow port 19999 through your firewall (e.g., sudo ufw allow 19999/tcp).
      url: step-2-configure-the-firewall
    - name: Access the Dashboard
      text: Open your web browser and navigate to http://your_server_ip:19999.
      url: step-3-access-your-dashboard
status: published
locale: en
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Command-line tools like [`htop` and `df`](/blog/monitor-vps-htop-df-free/) are excellent for quick troubleshooting when you are currently logged into an SSH session. But what if you need historical graphs? What if you want to see exactly how your CPU reacted when a burst of traffic hit your website 2 hours ago? 

For that, you need a full monitoring suite.

While enterprise teams rely on complex stacks like Prometheus and Grafana (which are tedious to set up and difficult to configure), there is a radically simpler, instantly beautiful alternative: **Netdata**.

Netdata installs in a single command, automatically detects all running services (like Nginx, Apache, MySQL, Docker), and instantly generates thousands of real-time metrics presented in a stunning web dashboard.

## Step 1: Install Netdata Using the Kickstart Script

Netdata provides an official, universally supported "kickstart" script. This handles identifying your OS architecture, downloading the required package dependencies, and installing the monitoring agent perfectly whether you are running Ubuntu, Debian, AlmaLinux, CentOS, or Fedora.

First, download the script to a temporary folder and execute it:

{% image "/assets/images/blog/en/setup-netdata-vps/H1.png", "Downloading and executing the Netdata kickstart installation script via wget on a Linux VPS", "(max-width: 768px) 100vw, 800px" %}

```bash
wget -O /tmp/netdata-kickstart.sh https://get.netdata.cloud/kickstart.sh && sh /tmp/netdata-kickstart.sh
```

The script will prompt you for confirmation. Press `Y` to confirm. 

It handles everything invisibly in the background. Once the installation is finished, Netdata automatically registers itself as a systemd service, starts running its daemons, and configures itself to boot whenever your server starts.

{% image "/assets/images/blog/en/setup-netdata-vps/H2.png", "Terminal output showing the successful installation of Netdata and its telemetry agents", "(max-width: 768px) 100vw, 800px" %}

To verify it is running smoothly, check the service status:

{% image "/assets/images/blog/en/setup-netdata-vps/H3.png", "Running sudo systemctl status netdata to verify the Netdata daemon is actively running in the background", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl status netdata
```

Look for `active (running)`.

## Step 2: Configure the Firewall

Netdata creates a lightweight web server strictly for serving its dashboard. By default, this web server listens on **Port 19999**.

Because you are likely (and should be!) running a firewall, port 19999 is blocked from the public internet. You need to explicitly open it so you can reach the dashboard from your browser.

**If you are using [UFW](/blog/configure-ufw-ubuntu-debian/) (Ubuntu/Debian):**
```bash
sudo ufw allow 19999/tcp
```

**If you are using [firewalld](/blog/configure-firewalld-centos-rhel/) (AlmaLinux/CentOS/Fedora):**
```bash
sudo firewall-cmd --permanent --add-port=19999/tcp
sudo firewall-cmd --reload
```

## Step 3: Access Your Dashboard

You are completely set up! 

Open your favorite web browser and navigate to your server's public IP address, appending the `:19999` port number.

`http://your_server_ip:19999`

{% image "/assets/images/blog/en/setup-netdata-vps/H4.png", "The beautifully graphed real-time Netdata visual monitoring dashboard loaded in a web browser on port 19999", "(max-width: 768px) 100vw, 800px" %}

You will immediately be loaded directly into the Netdata Local Dashboard. No passwords, no configurations, no waiting.

Scroll down the right-hand bar. Netdata will have already found and mapped graphs for:
- CPU usage by active core
- Hard drive I/O (read/write speeds)
- Total and available Memory (RAM) handling 
- Network bandwidth interfaces
- Interrupts, IPv4 tracking, and even background container (Docker) statistics.

### Security Note

By default, Netdata's local dashboard is accessible to anyone who has your server's IP address and knows to append `:19999`. While they cannot see your passwords or private code, they *can* map out what software you are running based on identifying the graphs (e.g., admitting you run MySQL to attackers).

If you are running a production server, it is highly recommended to eventually bind Netdata strictly to `localhost` and access it via a Reverse Proxy (using an Nginx Server Block) with a required password prompt (`htpasswd`). 

However, for a fresh testing or development environment, leaving the port open is fine for rapid monitoring. If you want to dive into complex performance metrics or monitor massive database loads efficiently, grab a remarkably robust [Budget VPS](/budget-vps/), spin up some intense applications, install Netdata, and watch the graphs dance perfectly in real-time.