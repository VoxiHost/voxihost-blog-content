---
image: /assets/images/blog/en/secure-ssh-centos-rhel/og-image.png
title: 'How to Secure SSH on AlmaLinux, CentOS, Rocky Linux & Fedora: The Complete Server Guide'
description: A complete guide to hardening SSH on AlmaLinux, CentOS Stream, Rocky Linux, and Fedora servers. Disable root login, set up key-based authentication, change the default port, configure firewalld, and protect your VPS against brute-force attacks.
date: '2026-03-25'
translationKey: secure-ssh-rhel-fedora
category: Tutorials
tags:
  - ssh
  - almalinux
  - centos
  - rocky linux
  - fedora
  - security
  - vps
  - hardening
  - key authentication
  - firewalld
  - brute force protection
howto:
  name: How to Secure SSH on AlmaLinux, CentOS Stream, Rocky Linux & Fedora
  totalTime: PT15M
  yield: A hardened RHEL-family or Fedora server with SSH key authentication, disabled root login, and firewalld-protected access
  tool:
    - A VPS or dedicated server running AlmaLinux, CentOS Stream, Rocky Linux, or Fedora
    - SSH client with key generation support (e.g. terminal, PuTTY)
    - sudo or root access
  steps:
    - name: Generate an SSH key pair
      text: Run ssh-keygen -t ed25519 on your local machine to generate a modern SSH key pair.
      url: generate-an-ssh-key-pair
    - name: Copy your public key to the server
      text: Run ssh-copy-id user@your-server-ip to install your public key on the server.
      url: copy-your-public-key-to-the-server
    - name: Disable root login
      text: Set PermitRootLogin no in /etc/ssh/sshd_config to prevent direct root access.
      url: disable-root-login
    - name: Disable password authentication
      text: Set PasswordAuthentication no in /etc/ssh/sshd_config to require key-based login only.
      url: disable-password-authentication
    - name: Change the default SSH port
      text: Set Port 2222 in sshd_config, update SELinux port labels, and configure firewalld accordingly.
      url: change-the-default-port
    - name: Restart SSH and verify
      text: Run sudo systemctl restart sshd and test your connection before closing the current session.
      url: restart-sshd-and-verify
status: published
locale: en
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

The moment a server with a public IP goes live, automated scanners start probing port 22. It's not personal, it's just what happens on the internet. Most of them are looking for root logins with weak passwords or default credentials from cloud images that haven't been touched.

Locking down SSH on AlmaLinux, CentOS Stream, Rocky Linux, and Fedora takes the same 15 minutes as on any Linux server, with one extra step that RHEL-based systems require: telling SELinux about any port changes you make. Skip that and you'll be wondering why SSH stopped working.

> **Prerequisite:** This guide disables root login. You **must** have a non-root user with `sudo` privileges ready **before** running any of these steps. If you haven't done that yet, follow our [How to Create a Sudo User on AlmaLinux, CentOS, Rocky Linux & Fedora](/blog/add-sudo-user-centos/) guide first, then come back here.

## Generate an SSH key pair

Do keys before anything else. Password authentication is the main vector for SSH brute-force attacks, and switching to keys eliminates it entirely.

On your **local machine**, generate an ed25519 key pair:

{% image "/assets/images/blog/en/secure-ssh-centos-rhel/H1.png", "Running ssh-keygen -t ed25519 -C \"your-server-label\" command on AlmaLinux, CentOS, Rocky Linux & Fedora to generate an ed25519 key pair", "(max-width: 768px) 100vw, 800px" %}

```bash
ssh-keygen -t ed25519 -C "your-server-label"
```

Set a passphrase when prompted. It encrypts the private key on disk, if someone gets your local machine, they still can't use the key without the passphrase.

## Copy your public key to the server

Copy the public key to the server:

{% image "/assets/images/blog/en/secure-ssh-centos-rhel/H2.png", "Running ssh-copy-id -i ~/.ssh/id_ed25519.pub user@your-server-ip command on AlmaLinux, CentOS, Rocky Linux & Fedora to copy the public key to the server", "(max-width: 768px) 100vw, 800px" %}

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@your-server-ip
```

Open a **new terminal window** and verify you can connect with the key before changing anything else. If you're in without a password prompt, the key works. Keep your original session open, you'll need it as a fallback if something goes wrong in later steps.

## Disable root login

Direct root login is an unnecessary risk. If your key gets compromised, an attacker immediately has unrestricted access. Use a non-root account with sudo instead.

Edit the SSH config:

{% image "/assets/images/blog/en/secure-ssh-centos-rhel/H3.png", "Running sudo nano /etc/ssh/sshd_config on AlmaLinux to open and edit the SSH daemon configuration file to disable root login", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo nano /etc/ssh/sshd_config
```

Find and set:

```ini
PermitRootLogin no
```

If this isn't set, on RHEL-based systems the default may vary by cloud image. Always set it explicitly.

## Disable password authentication

With your key confirmed working, disable passwords:

{% image "/assets/images/blog/en/secure-ssh-centos-rhel/H4.png", "Editing /etc/ssh/sshd_config on AlmaLinux to set PasswordAuthentication no and PubkeyAuthentication yes to enforce key-only login", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo nano /etc/ssh/sshd_config
```

Set:

```ini
PasswordAuthentication no
PubkeyAuthentication yes
```

On these distros, the main config file is usually authoritative. But double-check for overrides:

```bash
grep -r "PasswordAuthentication" /etc/ssh/
```

If anything in `/etc/ssh/sshd_config.d/` is setting it to `yes`, fix that file.

## Tighten a few more settings

Small changes that reduce exposure:

```ini
# Disconnect after 3 failed auth attempts
MaxAuthTries 3

# Reduce the window for incomplete connections
LoginGraceTime 30

# Disable features you're not using
X11Forwarding no
AllowTcpForwarding no
```

If only specific users should have SSH access:

```ini
AllowUsers youruser
```

Any system user not listed won't be able to authenticate remotely, even with valid credentials. Useful for keeping application accounts locked down.

## Change the default port

This is where RHEL-based systems differ from Debian-based ones. SELinux controls which ports services are allowed to listen on. If you change the SSH port without updating SELinux, the service will fail to restart.

First, check which ports SSH is currently allowed to use:

```bash
sudo semanage port -l | grep ssh
```

Add your new port to the allowed list:

```bash
sudo semanage port -a -t ssh_port_t -p tcp 2222
```

If `semanage` isn't available:

```bash
sudo dnf install policycoreutils-python-utils -y
```

Then edit `sshd_config`:

```ini
Port 2222
```

Now update firewalld to allow the new port and remove the old one:

```bash
sudo firewall-cmd --permanent --add-port=2222/tcp
sudo firewall-cmd --permanent --remove-service=ssh
sudo firewall-cmd --reload
```

Verify:

```bash
sudo firewall-cmd --list-all
```

You should see `2222/tcp` in the ports list and `ssh` removed from services.

## Restart sshd and verify

On RHEL-family systems the service is `sshd`, not `ssh`:

```bash
sudo systemctl restart sshd
```

In a **new terminal window**, connect on the new port:

```bash
ssh -p 2222 user@your-server-ip
```

If it works, you're done. If not, use your existing session to debug. Check for config syntax errors first:

```bash
sudo sshd -t
```

That command validates the config without actually restarting, it will tell you if there's a typo or invalid setting.

## Verify the SELinux port assignment

After restarting, confirm SELinux accepted the port:

```bash
sudo semanage port -l | grep ssh
```

You should see your new port listed. If the restart succeeded, this should already be fine.

## Check the auth logs

See what's hitting your server:

```bash
sudo journalctl -u sshd --since "1 hour ago" | grep -E "Failed|Invalid"
```

On a properly hardened server with password auth disabled and running on a non-standard port, this log should be essentially empty.

## SELinux audit denials

If sshd fails to start or connect after the port change, check for SELinux denials:

```bash
sudo ausearch -m avc -ts recent | grep sshd
```

That'll tell you exactly what SELinux blocked, which makes fixing it much simpler than guessing.

If you want a clean RHEL-based VPS to practice this on without risk, our [Budget VPS](/budget-vps/) plans are cheap enough to spin up a test box, harden it, and start fresh if anything breaks.