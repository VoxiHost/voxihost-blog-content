---
image: /assets/images/blog/en/secure-ssh-ubuntu-debian/og-image.png
title: 'How to Secure SSH on Ubuntu & Debian: The Complete Server Guide'
description: A complete guide to hardening SSH on Ubuntu and Debian servers. Disable root login, set up key-based authentication, change the default port, configure ufw, and lock down your VPS against brute-force attacks.
date: '2026-03-25'
translationKey: secure-ssh-ubuntu-debian
category: Tutorials
tags:
  - ssh
  - ubuntu
  - debian
  - security
  - vps
  - hardening
  - key authentication
  - ufw
  - brute force protection
howto:
  name: How to Secure SSH on Ubuntu & Debian
  totalTime: PT15M
  yield: A hardened Ubuntu or Debian server with SSH key authentication, disabled root login, and firewall-protected access
  tool:
    - A VPS or dedicated server running Ubuntu or Debian
    - SSH client with key generation support (e.g. terminal, PuTTY)
    - A non-root sudo user (see prerequisite below)
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
      text: Set PasswordAuthentication no in /etc/ssh/sshd_config to require key-based login.
      url: disable-password-authentication
    - name: Change the default SSH port
      text: Set Port 2222 in sshd_config and update your UFW firewall rules accordingly.
      url: change-the-default-port
    - name: Restart SSH and verify
      text: Run sudo systemctl restart ssh and verify from a new terminal window before closing your session.
      url: restart-ssh-and-verify
faq:
  - question: "Is changing the default SSH port really effective?"
    answer: "Yes. While it doesn't stop targeted hacks, changing the port from 22 to a custom port like 2222 stops 99% of automated scanners and script bots, keeping your <code>/var/log/auth.log</code> clean and reducing server CPU overhead."
  - question: "What is the difference between sshd_config and ssh_config?"
    answer: "The <code>sshd_config</code> file configures the SSH daemon (server-side, incoming connections), whereas <code>ssh_config</code> configures the SSH client (outgoing connections from your server)."
  - question: "What happens if I lose my SSH private key?"
    answer: "If password login is disabled and you lose your private key, you will be locked out. You must use your cloud provider's VNC or IPMI out-of-band console to log in and re-enable password auth or append a new public key."
  - question: "Why is UFW configuration critical before restarting SSH on a new port?"
    answer: "If you change the port to 2222 but do not run <code>sudo ufw allow 2222/tcp</code> before reloading the SSH service, UFW will block the new port, locking you out as soon as you disconnect."
  - question: "Can I allow only specific users to log in via SSH?"
    answer: "Yes, you can add <code>AllowUsers username1 username2</code> to your <code>/etc/ssh/sshd_config</code>. Any user not listed will be blocked from logging in, even if they have valid keys."
status: published
locale: en
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Port 22 is scanned constantly. The moment you spin up a VPS with a public IP, automated bots start hammering it for weak passwords and default credentials. Hardening SSH takes about 15 minutes and makes your server dramatically less interesting to anyone trying to get in.

> **Prerequisite:** This guide disables root login. You **must** have a non-root user with `sudo` privileges ready **before** running any of these steps. If you haven't done that yet, follow our [How to Create a Sudo User on Ubuntu & Debian](/blog/add-sudo-user-ubuntu/) guide first, then come back here.

## Generate an SSH key pair

The single most effective change you can make. Password logins can be brute-forced. Key-based auth cannot,  not in any realistic timeframe.

On your **local machine**, generate a key pair:

{% image "/assets/images/blog/en/secure-ssh-ubuntu-debian/H1.png", "Running ssh-keygen -t ed25519 -C \"your-server-label\" to generate a new SSH key pair", "(max-width: 768px) 100vw, 800px" %}

```bash
ssh-keygen -t ed25519 -C "your-server-label"
```

Use `ed25519`,  it's faster and more secure than the older RSA algorithm. When prompted for a passphrase, **set one**. It encrypts the private key on disk, so even if someone compromises your laptop, they still can't use the key without it.

## Copy your public key to the server

Copy the public key to the server. Replace `youruser` with your actual sudo username:

{% image "/assets/images/blog/en/secure-ssh-ubuntu-debian/H2.png", "Running ssh-copy-id -i ~/.ssh/id_ed25519.pub youruser@your-server-ip to copy the public key to the server", "(max-width: 768px) 100vw, 800px" %}

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub youruser@your-server-ip
```

**Test the key works before moving on.** Open a new terminal window and connect. If you get in without a password prompt, the key is installed correctly. **Do not close your existing session yet**,  you still need to disable password auth as a separate step.

Optional: add an entry to `~/.ssh/config` on your local machine for quick access:

{% image "/assets/images/blog/en/secure-ssh-ubuntu-debian/H3.png", "Fast connection to the server", "(max-width: 768px) 100vw, 800px" %}

```
Host myserver
    HostName your-server-ip
    User youruser
    IdentityFile ~/.ssh/id_ed25519
```

{% image "/assets/images/blog/en/secure-ssh-ubuntu-debian/H4.png", "Fast connection to the server", "(max-width: 768px) 100vw, 800px" %}

After this, `ssh myserver` is all you need to type.


## Disable root login

SSH in as your sudo user from this point forward. Direct root login is a security risk,  if your session is compromised, an attacker has full unrestricted access with zero additional steps.

Open the SSH daemon config:

{% image "/assets/images/blog/en/secure-ssh-ubuntu-debian/H5.png", "Disabling root login in sshd_config", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo nano /etc/ssh/sshd_config
```

Find and update this line:

```ini
PermitRootLogin no
```

If you ever need root access on the server, SSH in as your sudo user and run `sudo su` from there.

## Disable password authentication

Your key is working, so now eliminate password logins entirely:

```bash
sudo nano /etc/ssh/sshd_config
```

Set both of these:

```ini
PasswordAuthentication no
PubkeyAuthentication yes
```

> **Important:** Some Ubuntu and Debian versions set `PasswordAuthentication` in a drop-in file under `/etc/ssh/sshd_config.d/` that **overrides** the main config. Check for it:
> ```bash
> grep -r "PasswordAuthentication" /etc/ssh/
> ```
> If you see it set to `yes` anywhere in the output, edit that specific file,  not the main `sshd_config`.

## Tighten a few more settings

While you have `sshd_config` open, add these to reduce the attack surface further:

```ini
# Disconnect after 3 failed attempts
MaxAuthTries 3

# Close unauthenticated connections faster
LoginGraceTime 30

# Disable unused features
X11Forwarding no
AllowTcpForwarding no
```

If only specific usernames should be able to log in remotely, add an allowlist:

```ini
AllowUsers youruser
```

Any account not on the list will be refused at the SSH level, even with a valid key.

## Change the default port

Port 22 appears in every scanner's default target list. Moving SSH to a non-standard port won't stop a determined attacker from port-scanning, but it eliminates virtually all the automated noise. Auth logs go from hundreds of failed login attempts per day to effectively zero.

In `sshd_config`, update the port:

```ini
Port 2222
```

Choose any unused port above 1024. **Before restarting SSH**, update your firewall to allow the new port and close the old one:

```bash
sudo ufw allow 2222/tcp
sudo ufw deny 22/tcp
sudo ufw status
```

Make sure **2222 shows as ALLOW** in the output before proceeding.

## Restart SSH and verify

Apply all your changes:

```bash
sudo systemctl restart ssh
```

Then, in a **new terminal window** (keep your current session open), test the connection on the new port:

```bash
ssh -p 2222 youruser@your-server-ip
```

If it connects cleanly, you're done. If it fails, return to your existing session and debug. Run `sudo sshd -t` to check `sshd_config` for syntax errors before restarting again.

Common issues:
- Firewall not updated for the new port
- `PasswordAuthentication no` set in a drop-in file that was missed

## Check what the server is actually seeing

After locking things down, inspect live authentication attempts:

```bash
sudo journalctl -u ssh --since "1 hour ago" | grep "Failed"
```

On a properly hardened server you should see nothing,  or just a handful of attempts on the old port being silently dropped by the firewall.

## A note on fail2ban

With SSH key auth enabled and password auth disabled, brute-force attacks against SSH are already impossible. fail2ban becomes less critical for SSH itself. That said, it's still useful for protecting other services like Nginx and Apache, and running it alongside these settings adds a reasonable layer of defense in depth. See our [fail2ban setup guide](/blog/setup-fail2ban-ubuntu-debian/) if you want to add it.

If you want a safe place to practice this hardening process without risking a production server, our **[Budget VPS](/budget-vps/)** plans are an affordable sandbox to lock down, break, and start fresh as many times as you need.