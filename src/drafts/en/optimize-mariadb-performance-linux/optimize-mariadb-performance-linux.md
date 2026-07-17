---
image: /assets/images/blog/en/optimize-mariadb-performance-linux/og-image.png
title: "How to Optimize MariaDB Performance on Linux"
description: "Learn how to optimize MariaDB performance on Linux by tuning system settings, disabling Transparent Huge Pages, and configuring InnoDB for your VPS."
status: draft
category: Tutorials
tags:
  - mariadb
  - linux
  - mysql
  - server
  - vps
  - hosting
date: '2026-07-23'
locale: en
translationKey: optimize-mariadb-performance-linux
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors: []
howto:
  name: "How to Optimize MariaDB Performance on Linux"
  steps:
    - name: "Step 1: Install MariaDB from the Official Repository"
      url: "step-1-install-mariadb-from-the-official-repository"
    - name: "Step 2: Optimize System-Level Performance"
      url: "step-2-optimize-system-level-performance"
    - name: "Step 3: Disable Transparent Huge Pages"
      url: "step-3-disable-transparent-huge-pages"
    - name: "Step 4: Configure MariaDB InnoDB Settings"
      url: "step-4-configure-mariadb-innodb-settings"
faq:
  - question: "Why does MariaDB require a restart after changing innodb_buffer_pool_size?"
    answer: "The <code>innodb_buffer_pool_size</code> defines the memory area where data and indexes are cached. Because this memory is allocated at startup, <strong>VoxiHost</strong> recommends a service restart to ensure the database correctly initializes the new memory allocation."
  - question: "How can I verify that Transparent Huge Pages are disabled on my server?"
    answer: "You can verify the status by running <code>cat /sys/kernel/mm/transparent_hugepage/enabled</code>. If the output shows <code>[never]</code>, the feature is successfully disabled."
  - question: "What is the recommended swappiness value for a database server?"
    answer: "For production database workloads, we recommend a <code>swappiness</code> value of <strong>10</strong>. This forces the kernel to favor keeping data in physical RAM rather than moving it to disk."
  - question: "Does the change in innodb_snapshot_isolation affect legacy applications?"
    answer: "Yes, the default change to <code>ON</code> in MariaDB 12.3 might alter transaction behavior for older applications. Always test your application in a staging environment on a <strong>Premium VPS</strong> before applying these changes to production."
  - question: "How do I check if my MariaDB service is using the latest LTS release?"
    answer: "You can check your installed version by running <code>mariadb --version</code>. If you followed the official repository setup, <code>apt update && apt upgrade</code> will ensure you remain on the latest stable LTS branch."
---

Database performance often dictates the ceiling of your application speed. While a fresh installation of MariaDB works out of the box, default configurations are designed for compatibility rather than raw throughput. On a <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Premium VPS](/premium-vps/), you have the underlying power to handle thousands of concurrent queries, but inefficient memory management or system-level bottlenecks can quickly negate that advantage.

Optimizing MariaDB requires a layered approach. You cannot simply adjust a few buffer settings and expect results; you must tune the host operating system to stop fighting against the database engine. This includes managing how the kernel handles memory allocation and ensuring that file descriptor limits are high enough to support heavy connection loads. 

We will implement performance-oriented system tuning, disable conflicting kernel features like Transparent Huge Pages, and install the latest stable version of MariaDB directly from the official repositories. Your database will be configured to handle demanding production workloads with significantly lower latency and improved stability. Whether you are running a high-traffic web application or a resource-heavy data service, these changes will provide the foundation for a lean, efficient database environment.

{% image "/assets/images/blog/en/optimize-mariadb-performance-linux/H1.png", "A terminal window showing a successful MariaDB service status check on a Linux server", "(max-width: 768px) 100vw, 800px" %}

## Prerequisites

Before we begin the optimization process, ensure your environment meets the necessary technical baseline. You should have a server running either Ubuntu 22.04 or Debian 12. While MariaDB can run on lower specifications, we strongly recommend a minimum of 2GB of RAM and 2 CPU cores to avoid resource contention during high-traffic periods. 

Access to your server must be via a user account with sudo privileges. If you are using a <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Premium VPS](/premium-vps/) or [Budget VPS](/budget-vps/), you likely already have this setup. Ensure that your firewall is configured to allow database connections only from trusted sources. If you have not yet secured your instance, you should review our guide on [securing a Linux server firewall](/securing-linux-server-firewall/) before opening any database ports to the public.

Finally, verify that your system is up to date. You will need basic administrative utilities installed to follow the configuration steps provided in this guide. If you are starting from a clean slate, ensure your package list is refreshed and your system clock is synchronized to prevent issues with transaction logging.

{% image "/assets/images/blog/en/optimize-mariadb-performance-linux/H2.png", "Terminal output displaying system requirements check for a Linux database server", "(max-width: 768px) 100vw, 800px" %}

## Step 1: Install MariaDB from the Official Repository

To ensure you have access to the latest LTS releases and security patches, we will pull MariaDB directly from the official repository. Using the default system repositories often provides an outdated version, which can lead to compatibility issues with modern performance features.

The MariaDB team provides a setup script that automatically detects your distribution and configures the correct repository files. Execute the following commands to add the official repository and install the necessary database packages:

```bash
## Add the official MariaDB repository and update your package lists
curl -LsS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash
sudo apt update

## Install the MariaDB server and client packages
sudo apt install -y mariadb-server mariadb-client
```

Once the installation completes, the database service will start automatically. You can verify that the service is active and running correctly by checking its status:

```bash
## Confirm the MariaDB service is currently active
systemctl status mariadb
```

To secure your installation, run the provided security script. This script removes anonymous users, disables remote root logins, and removes test databases:

```bash
## Run the security installation script
sudo mysql_secure_installation
```

{% image "/assets/images/blog/en/optimize-mariadb-performance-linux/H3.png", "Terminal output showing the successful installation and security script execution of the MariaDB server", "(max-width: 768px) 100vw, 800px" %}

## Step 2: Optimize System-Level Performance

Before tuning the MariaDB configuration files, we must address how the operating system manages memory and file resources. Default Linux settings are often too generic for database workloads, which can lead to unnecessary disk swapping or file descriptor bottlenecks when your <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> server experiences high traffic.

Next, we need to adjust the kernel's swap behavior and increase the file descriptor limits. By lowering the `vm.swappiness` value, we instruct the system to prioritize keeping the database in RAM rather than moving it to disk. We also increase the limit of open files to ensure the server can handle a high volume of concurrent connections without hitting system caps.

```bash
## Set swappiness to 10 to reduce disk swapping
echo "vm.swappiness=10" | sudo tee /etc/sysctl.d/99-mariadb-tuning.conf
sudo sysctl --system

## Increase the file descriptor limits for better concurrency
echo -e "* soft nofile 65536\n* hard nofile 65536" | sudo tee -a /etc/security/limits.conf
```

These changes ensure that the underlying Linux environment is optimized to support high-performance database operations. By reducing the likelihood of the kernel offloading memory to swap, your database will remain responsive even during heavy query loads.

{% image "/assets/images/blog/en/optimize-mariadb-performance-linux/H4.png", "Terminal output showing the application of sysctl tuning and file descriptor limits", "(max-width: 768px) 100vw, 800px" %}

## Step 3: Disable Transparent Huge Pages

We must specifically target Transparent Huge Pages (THP). While THP aims to simplify memory management for large-scale applications, its aggressive memory allocation strategy often conflicts with the way InnoDB manages its buffer pool.

When THP is active, the Linux kernel can cause significant latency spikes during database operations, as the system attempts to collapse memory pages into huge pages. For a database server running on a <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Premium VPS](/premium-vps/), disabling this feature is a standard best practice to ensure predictable performance and minimize internal memory contention.

To make this configuration persistent across reboots, we create a systemd service file:

```bash
## Create a systemd service to disable THP on boot
cat <<EOF | sudo tee /etc/systemd/system/disable-thp.service
[Unit]
Description=Disable Transparent Huge Pages (THP)
After=sysinit.target

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'echo never > /sys/kernel/mm/transparent_hugepage/enabled && echo never > /sys/kernel/mm/transparent_hugepage/defrag'

[Install]
WantedBy=multi-user.target
EOF

## Enable and start the service
sudo systemctl daemon-reload
sudo systemctl enable --now disable-thp.service
```

{% image "/assets/images/blog/en/optimize-mariadb-performance-linux/H5.png", "Terminal showing the creation of a systemd service to disable transparent huge pages", "(max-width: 768px) 100vw, 800px" %}

## Step 4: Configure MariaDB InnoDB Settings

The heart of MariaDB performance lies within the InnoDB storage engine. The `innodb_buffer_pool_size` is the most critical variable to adjust. It dictates how much memory is allocated for caching data and indexes. For a dedicated database server, a common rule of thumb is to set this to 70-80% of your total available system RAM.

Open your configuration file to begin the tuning process:

```bash
## Open the main MariaDB server configuration file
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Locate the `[mysqld]` section and add or adjust the following directives. Ensure you replace `[RAM_SIZE]` with a value appropriate for your <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Premium VPS](/premium-vps/) instance:

```ini
[mysqld]
## Set the buffer pool to 75% of total system RAM
innodb_buffer_pool_size = 6G
## Ensure logs are flushed to disk efficiently
innodb_flush_log_at_trx_commit = 2
## Optimize the number of I/O threads
innodb_write_io_threads = 8
innodb_read_io_threads = 8
```

> **Warning:** Changing `innodb_buffer_pool_size` requires a full service restart to take effect. If you set this value too high, you risk triggering the system OOM (Out of Memory) killer, which will crash your database.

After saving your changes with `Ctrl+O` and `Enter`, and exiting with `Ctrl+X`, restart the service to apply the new configuration:

```bash
## Restart MariaDB to apply the new InnoDB memory settings
sudo systemctl restart mariadb
```

The `innodb_flush_log_at_trx_commit = 2` setting provides a massive performance boost for write-heavy applications by flushing logs to the OS cache every second rather than on every individual commit. This is safe for most web applications, though you should revert to `1` if your workload demands strict ACID compliance for every transaction.

{% image "/assets/images/blog/en/optimize-mariadb-performance-linux/H6.png", "Editing the MariaDB server configuration file to adjust InnoDB buffer pool settings", "(max-width: 768px) 100vw, 800px" %}

## Conclusion

Optimizing MariaDB is an iterative process. By disabling Transparent Huge Pages, tuning system-level parameters like swappiness, and properly sizing your InnoDB buffer pool, you have established a solid foundation for high-performance database operations. These adjustments ensure your <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> server manages resources efficiently, reducing latency and preventing bottlenecks under heavy traffic.

Remember that database performance is rarely static. As your dataset grows or application usage patterns shift, you should periodically monitor your metrics using tools like `htop` or the `SHOW ENGINE INNODB STATUS;` command within the MariaDB client. Keeping your server updated is equally vital; always verify the latest stable releases to benefit from ongoing upstream performance patches.

Regular maintenance of your [Premium VPS](/premium-vps/) will keep your services running smoothly. For those managing more complex web environments, you may also find our guide on how to [Set Up a LEMP Stack (Linux, Nginx, MariaDB, PHP) on Ubuntu & Debian](/install-lemp-ubuntu-debian/) useful for integrating your optimized database with a robust web server. Proper configuration now saves significant troubleshooting time later.

{% image "/assets/images/blog/en/optimize-mariadb-performance-linux/H7.png", "A terminal showing the successful restart of the MariaDB service after performance tuning", "(max-width: 768px) 100vw, 800px" %}
