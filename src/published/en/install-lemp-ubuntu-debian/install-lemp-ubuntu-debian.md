---
image: /assets/images/blog/en/install-lemp-ubuntu-debian/og-image.png
title: How to Set Up a LEMP Stack (Linux, Nginx, MariaDB, PHP) on Ubuntu & Debian
description: A complete step-by-step guide to installing the modern LEMP stack (Linux, Nginx, MariaDB, PHP-FPM) on a fresh Ubuntu or Debian server.
date: '2026-03-25'
updated: '2026-06-02'
translationKey: setup-lemp-stack-ubuntu-debian
category: Tutorials
tags:
  - lemp
  - nginx
  - mariadb
  - php
  - ubuntu
  - debian
  - linux
  - vps
  - web server
howto:
  name: How to Install a LEMP Stack on Ubuntu or Debian
  totalTime: PT15M
  yield: A fully functioning web server capable of rendering dynamic PHP applications backed by a MariaDB database
  tool:
    - A VPS or dedicated server running Ubuntu or Debian
    - SSH client (e.g. terminal, PuTTY)
    - A user account with sudo privileges
  steps:
    - name: Install Nginx (The Web Server)
      text: Run sudo apt install nginx and allow it through your firewall.
      url: step-1-install-nginx-the-web-server
    - name: Install MariaDB (The Database)
      text: Run sudo apt install mariadb-server and secure it using mysql_secure_installation.
      url: step-2-install-mariadb-the-database
    - name: Install PHP (The Processing Language)
      text: Run sudo apt install php-fpm php-mysql to install the FastCGI Process Manager.
      url: step-3-install-php-the-processing-language
    - name: Configure Nginx to use PHP
      text: Edit your Nginx Server Block to pass .php files to the PHP-FPM socket.
      url: step-4-configure-nginx-to-use-php
    - name: Test your LEMP Stack
      text: Create an info.php file to verify Nginx and PHP are communicating correctly.
      url: step-5-test-php-processing-on-nginx
faq:
  - question: "What is a LEMP stack and how does it differ from LAMP?"
    answer: "A LEMP stack is a web software bundle where the <b>E</b> stands for Nginx (pronounced <i>Engine-X</i>) instead of Apache. Nginx is event-driven and offers better performance under high concurrent loads compared to Apache."
  - question: "Why is MariaDB preferred over MySQL in LEMP?"
    answer: "MariaDB is a community-developed, binary drop-in replacement for MySQL. It is fully open-source, offers faster query processing and better performance optimizations out of the box, and is the default database package in Debian and modern Ubuntu versions."
  - question: "Why does Nginx require PHP-FPM instead of a standard PHP module?"
    answer: "Unlike Apache, Nginx cannot run PHP code directly within its web server process. It relies on <b>PHP-FPM</b> (FastCGI Process Manager) to handle PHP processing externally. Nginx acts as a reverse proxy, passing PHP requests to the PHP-FPM daemon via a Unix socket."
  - question: "How do I find which PHP-FPM version and socket path Nginx is using?"
    answer: "You can find your PHP-FPM version by running <code>php -v</code>. The socket file is typically located in the <code>/var/run/php/</code> directory (e.g., <code>/var/run/php/php8.3-fpm.sock</code>). You must match this socket path exactly in your Nginx server block configuration."
  - question: "What should I do if Nginx returns a 502 Bad Gateway error when loading PHP files?"
    answer: "A 502 Bad Gateway error usually means Nginx cannot communicate with PHP-FPM. Check that the PHP-FPM service is running (<code>sudo systemctl status phpX.X-fpm</code>) and verify that the socket path in your Nginx configuration matches the active PHP-FPM configuration."
status: published
locale: en
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

The **LEMP** stack is the modern, high-performance foundation for millions of websites worldwide. It is an acronym representing the four critical pieces of software required to host dynamic, database-driven applications (like WordPress or Laravel):

- **L**inux: The operating system (Ubuntu or Debian).
- **E**Nginx (pronounced *Engine-X*): The lightning-fast web server.
- **M**ariaDB: The community-driven, drop-in replacement for MySQL.
- **P**HP: The backend processing language.

Compared to the older LAMP (Apache) stack, LEMP is highly favored for environments handling heavy, concurrent traffic because of Nginx's asynchronous architecture.

## Step 1: Install Nginx (The Web Server)

First, update your package index to ensure you are downloading the latest software versions.

```bash
sudo apt update
sudo apt upgrade -y
```

Install Nginx:

{% image "/assets/images/blog/en/install-lemp-ubuntu-debian/H1.png", "Running sudo apt install nginx -y on Ubuntu or Debian to install Nginx as part of the LEMP stack", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt install nginx -y
```

If you followed our [UFW Firewall Guide](/blog/configure-ufw-ubuntu-debian/), you need to allow Nginx traffic through the firewall. Open both HTTP (Port 80) and HTTPS (Port 443):

```bash
sudo ufw allow 'Nginx Full'
```

You can verify Nginx is running by typing your server's public IP address into your web browser. You should see the standard *"Welcome to nginx!"* page.

## Step 2: Install MariaDB (The Database)

Now that you have a web server, you need a database system to store and manage your application's data. **MariaDB** is a highly optimized, fully open-source fork of MySQL that is standard on modern Linux distributions.

Install the MariaDB server:

{% image "/assets/images/blog/en/install-lemp-ubuntu-debian/H2.png", "Running sudo apt install mariadb-server mariadb-client -y on Ubuntu to install MariaDB as part of the LEMP stack", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt install mariadb-server -y
```

Once installed, the database is active but completely unsecured. You need to lock it down using the built-in security script.

{% image "/assets/images/blog/en/install-lemp-ubuntu-debian/H3.png", "Running sudo mysql_secure_installation on Ubuntu to remove anonymous users, disable remote root login, and secure MariaDB", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo mysql_secure_installation
```

You will be asked a series of prompts:
1. **Current root password**: Press `Enter` (there is no password yet).
2. **Switch to unix_socket authentication**: Type `Y`.
3. **Change the root password**: Type `N` (modern MariaDB secures the root user dynamically using your Linux `sudo` privileges).
4. **Remove anonymous users**: Type `Y`.
5. **Disallow root login remotely**: Type `Y`.
6. **Remove test database**: Type `Y`.
7. **Reload privilege tables**: Type `Y`.

Your database is now locked down and ready.

## Step 3: Install PHP (The Processing Language)

Nginx is incredibly fast at serving static files (HTML, images, CSS), but it cannot process dynamic PHP code natively the way Apache can. 

To process PHP, we must install **PHP-FPM** (FastCGI Process Manager). Nginx will pass all `.php` files it receives directly to this background processor. You also need the `php-mysql` package so PHP can talk to your MariaDB database.

Install both packages:

{% image "/assets/images/blog/en/install-lemp-ubuntu-debian/H4.png", "Running sudo apt install php-fpm php-mysql to install PHP 8 with FPM and MySQL extension on Ubuntu for LEMP stack", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt install php-fpm php-mysql -y
```

*Note: Depending on your exact Debian/Ubuntu version, `apt` will automatically install the correct PHP version (e.g., `php8.1-fpm` or `php8.3-fpm`). Make a mental note of which version it installs, as you'll need it in the next step.*

## Step 4: Configure Nginx to use PHP

We need to explicitly tell Nginx how to handle PHP files. 

Let's assume you are configuring the default Nginx Server Block. Open the default configuration file in nano:

{% image "/assets/images/blog/en/install-lemp-ubuntu-debian/H5.png", "Creating a new Nginx server block configuration file in sites-available for a custom domain on Ubuntu LEMP setup", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo nano /etc/nginx/sites-available/default
```

Look for the `index` directive. You need to add `index.php` to the very beginning of the list, telling Nginx to prioritize PHP files over standard HTML files.

```nginx
# Add index.php before index.html
index index.php index.html index.htm index.nginx-debian.html;
```

Next, scroll down to the `location ~ \.php$` block. Uncomment (remove the `#` symbol) from the relevant lines so it looks exactly like this. **Make sure the `phpX.X-fpm.sock` matches the version you installed in Step 3!**

```nginx
location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
}
```

Save the file and exit `nano`.

Test your Nginx configuration for syntax errors:

{% image "/assets/images/blog/en/install-lemp-ubuntu-debian/H6.png", "Running sudo nginx -t to test Nginx server block configuration for syntax errors before enabling the LEMP site", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo nginx -t
```

If it reports `syntax is ok`, reload Nginx to apply the changes:
```bash
sudo systemctl reload nginx
```

## Step 5: Test PHP Processing on Nginx

To prove that Nginx is successfully handing off code to PHP-FPM, we will create a classic PHP info script.

Create a new file in your web root directory:

{% image "/assets/images/blog/en/install-lemp-ubuntu-debian/H7.png", "Creating a PHP info test file at /var/www/your_domain/info.php with nano to verify PHP-FPM on Nginx LEMP stack", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo nano /var/www/html/info.php
```

Paste the following PHP code:
```php
<?php
phpinfo();
?>
```

Save and exit.

Open your browser and navigate to: `http://your_server_ip/info.php`

{% image "/assets/images/blog/en/install-lemp-ubuntu-debian/H8.png", "phpinfo() page rendering in browser confirming PHP is working correctly with Nginx and MariaDB on the LEMP stack", "(max-width: 768px) 100vw, 800px" %}

You should see a massive, detailed purple and gray table outlining your server's exact PHP configuration and modules. This proves your complete LEMP stack is functioning perfectly! 

**Crucial Warning:** Delete this file immediately. Leaving it publicly accessible exposes extremely sensitive information about your server's configuration to hackers.

```bash
sudo rm /var/www/html/info.php
```

Your server is now a fully powered production machine. Ready to host millions of hits? Deploy an intensely fast [Premium VPS](/premium-vps/), install your LEMP stack, get a [free Let's Encrypt SSL](/blog/ssl-letsencrypt-certbot-ubuntu/), and launch your database-driven application directly to the world.