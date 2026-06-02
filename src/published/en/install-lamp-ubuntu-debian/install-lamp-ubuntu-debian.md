---
image: /assets/images/blog/en/install-lamp-ubuntu-debian/og-image.png
title: How to Set Up a LAMP Stack (Linux, Apache, MySQL, PHP) on Ubuntu & Debian
description: A comprehensive beginner-friendly guide to installing the tried-and-true LAMP stack (Linux, Apache2, MySQL, PHP) on Ubuntu and Debian.
date: '2026-03-25'
updated: '2026-06-02'
translationKey: setup-lamp-stack-ubuntu-debian
category: Tutorials
tags:
  - lamp
  - apache
  - mysql
  - php
  - ubuntu
  - debian
  - linux
  - vps
  - web server
howto:
  name: How to Install a LAMP Stack on Ubuntu or Debian
  totalTime: PT15M
  yield: A fully functioning web server capable of rendering dynamic PHP applications using Apache and MySQL
  tool:
    - A VPS or dedicated server running Ubuntu or Debian
    - SSH client (e.g. terminal, PuTTY)
    - A user account with sudo privileges
  steps:
    - name: Install Apache (The Web Server)
      text: Run sudo apt install apache2 and configure UFW to allow Apache Full traffic.
      url: step-1-install-apache-the-web-server
    - name: Install MySQL (The Database)
      text: Run sudo apt install mysql-server and secure it using the built-in MySQL security script.
      url: step-2-install-mysql-the-database
    - name: Install PHP
      text: Run sudo apt install php libapache2-mod-php php-mysql to install PHP and tie it directly into Apache.
      url: step-3-install-php
    - name: Configure Apache's Directory Index
      text: Edit dir.conf to prioritize index.php over standard index.html files.
      url: step-4-configure-apache-index-priorities
    - name: Test PHP Processing
      text: Create an info.php file in /var/www/html to verify your configuration.
      url: step-5-test-the-lamp-stack
faq:
  - question: "What is a LAMP stack?"
    answer: "A LAMP stack is a classic open-source software bundle used for hosting dynamic websites and web applications. It stands for <b>L</b>inux (operating system), <b>A</b>pache (web server), <b>M</b>ySQL (database), and <b>P</b>HP (programming language)."
  - question: "Why do we need the libapache2-mod-php package?"
    answer: "The <code>libapache2-mod-php</code> package is an Apache module that allows the Apache web server to directly run and process PHP files within its own processes, without needing a separate PHP-FPM service."
  - question: "How do I secure the MySQL installation after installing the package?"
    answer: "You should run the built-in security script <code>sudo mysql_secure_installation</code>, which helps you set a root password, remove anonymous users, disable remote root logins, and drop the test database."
  - question: "Why is it important to change the Directory Index priority in dir.conf?"
    answer: "By default, Apache looks for <code>index.html</code> files first. If your web application relies on PHP (like WordPress), prioritizing <code>index.php</code> in <code>dir.conf</code> ensures that visitors are directed to the dynamic PHP home page immediately."
  - question: "Why should I delete the info.php test file after verifying the setup?"
    answer: "The <code>info.php</code> file exposes detailed information about your server's configuration, PHP version, modules, and path variables. Leaving it public is a security risk as malicious actors can exploit it."
status: published
locale: en
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

The **LAMP** stack is the undisputed grandfather of open-source web hosting. For decades, it has been the most reliable, thoroughly documented, and widespread foundation for hosting dynamic web applications like WordPress, Drupal, and Joomla.

LAMP stands for four components:
- **L**inux: The operating system (Ubuntu or Debian).
- **A**pache: The incredibly robust, highly customizable web server.
- **M**ySQL: The most popular relational database management system.
- **P**HP: The server-side scripting language handling the backend logic.

Compared to the newer LEMP (Nginx) stack, LAMP remains universally beloved because Apache processes PHP dynamically natively (no need to configure external FPM sockets) and relies on highly flexible `.htaccess` files for easy, per-directory configuration overrides.

## Step 1: Install Apache (The Web Server)

Before installing any software, always update your local package indexes so you are downloading the latest security patches.

```bash
sudo apt update
sudo apt upgrade -y
```

Now, install the Apache web server (the package is named `apache2` on Debian/Ubuntu systems):

{% image "/assets/images/blog/en/install-lamp-ubuntu-debian/H1.png", "Running sudo apt install apache2 -y to start Apache installation", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt install apache2 -y
```

If you are running the `ufw` firewall (which you should be, per our [Security Guide](/blog/configure-ufw-ubuntu-debian/)), you need to allow Apache traffic to pass through. You want to open both HTTP (Port 80) and HTTPS (Port 443).

```bash
sudo ufw allow 'Apache Full'
```

To verify your web server is alive, type your server's public IP address into your favorite web browser (`http://your_server_ip`). You should see the default *"Apache2 Ubuntu Default Page"*.

## Step 2: Install MySQL (The Database)

Your server can serve static HTML now, but to store application data (like user accounts, blog posts, and settings), you need a database.

Install the official MySQL server:

{% image "/assets/images/blog/en/install-lamp-ubuntu-debian/H2.png", "Running sudo apt install mysql-server -y to start MySQL installation", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt install mysql-server -y
```

Once the installation finishes, the database is running but its default configuration is dangerously open. You must lock it down using the interactive security script:

{% image "/assets/images/blog/en/install-lamp-ubuntu-debian/H3.png", "Running sudo mysql_secure_installation to secure MySQL installation", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo mysql_secure_installation
```

You will be asked several questions to configure the security profile:
1. **Validate Password Plugin**: Type `y` if you want MySQL to actively block weak passwords, or `n` to skip it.
2. **Remove anonymous users**: Type `y`.
3. **Disallow root login remotely**: Type `y` (root should only ever access the database from *inside* the server).
4. **Remove test database**: Type `y`.
5. **Reload privilege tables**: Type `y`.

MySQL is now secure.

## Step 3: Install PHP

You have a web server and a database, but they cannot communicate with each other yet, nor can they process dynamic code. You need PHP.

For Apache, installing PHP requires three main packages: the core `php` package, the `php-mysql` extension allowing PHP scripts to talk to your database, and the vital `libapache2-mod-php` package, which magically binds PHP processing directly into the Apache runtime.

{% image "/assets/images/blog/en/install-lamp-ubuntu-debian/H4.png", "Running sudo apt install php libapache2-mod-php php-mysql -y to start PHP installation", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt install php libapache2-mod-php php-mysql -y
```

## Step 4: Configure Apache Index Priorities

When a user visits a directory on your website (like `yoursite.com/blog/`), Apache automatically looks for a default "index" file to serve. By default, it looks for `index.html` first, and if it doesn't find it, it eventually looks for `index.php`.

For dynamic applications, we want Apache to prioritize `index.php`. 

{% image "/assets/images/blog/en/install-lamp-ubuntu-debian/H5.png", "Running sudo nano /etc/apache2/mods-enabled/dir.conf to open the dir.conf file", "(max-width: 768px) 100vw, 800px" %}

Open the `dir.conf` file in the nano text editor:
```bash
sudo nano /etc/apache2/mods-enabled/dir.conf
```

It will look like this:
```apache
<IfModule mod_dir.c>
    DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
</IfModule>
```

Move the `index.php` string from the middle of the list directly to the first position, immediately after `DirectoryIndex`, so it looks like this:
```apache
<IfModule mod_dir.c>
    DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```

Save and exit `nano`. 

Whenever you alter Apache's configuration modules, you inherently must restart the web server for the changes to take effect:
```bash
sudo systemctl restart apache2
```

## Step 5: Test the LAMP Stack

Your environment is complete! However, the golden rule of system administration is to verify your work. We are going to write a tiny PHP script to prove that Apache can process dynamic code.

Create a new file in Apache's default web root directory:
```bash
sudo nano /var/www/html/info.php
```

Paste the standard initialization function:
```php
<?php
phpinfo();
?>
```

Save the file. Open your web browser and navigate to `http://your_server_ip/info.php`.

{% image "/assets/images/blog/en/install-lamp-ubuntu-debian/H6.png", "Running sudo systemctl status apache2 on Ubuntu to verify Apache is active after completing the full LAMP stack installation", "(max-width: 768px) 100vw, 800px" %}

If the installation was successful, you will be greeted by a massive, highly detailed table detailing your PHP version, installed modules, memory limits, and Apache integration settings. 

**Critical Security Warning:** The `info.php` page contains an extensive roadmap of your internal server architecture. Leaving this file public is a massive security risk. Once you have confirmed the stack works, **delete the file immediately**:

```bash
sudo rm /var/www/html/info.php
```

Congratulations! You have successfully built a tried, tested, and rock-solid foundation for hosting on Linux. Do you have a heavy e-commerce site, forum, or high-traffic blog ready to launch? Pair your new LAMP stack with one of our high-tier [Premium VPS](/premium-vps/) environments or a highly cost-effective [Budget VPS](/budget-vps/), install a [free SSL via Certbot](/blog/ssl-letsencrypt-certbot-ubuntu/), and build the ultimate web experience.