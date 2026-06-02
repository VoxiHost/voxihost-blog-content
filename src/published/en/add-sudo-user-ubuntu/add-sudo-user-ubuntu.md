---
image: /assets/images/blog/en/add-sudo-user-ubuntu/og-image.png
title: 'How to Create a Sudo User on Ubuntu & Debian: The Complete Server Guide'
description: A complete beginner-friendly guide to creating a new user with sudo privileges on Ubuntu and Debian servers. Stop logging in as root and secure your VPS.
date: '2026-03-25'
translationKey: create-sudo-user-ubuntu-debian
category: Tutorials
tags:
  - ubuntu
  - debian
  - sudo
  - user management
  - linux
  - vps
  - security
  - usermod
  - adduser
howto:
  name: How to Create a Sudo User on Ubuntu and Debian
  totalTime: PT5M
  yield: A new non-root user account with full administrative (sudo) privileges
  tool:
    - A VPS or dedicated server running Ubuntu or Debian
    - SSH client (e.g. terminal, PuTTY)
    - Access to the root account
  steps:
    - name: Log in as root
      text: Connect to your server via SSH using the root account.
      url: step-1-log-in-as-root
    - name: Add the new user
      text: Run adduser username and follow the prompts to set a strong password.
      url: step-2-add-the-new-user
    - name: Grant sudo privileges
      text: Run usermod -aG sudo username to add the new user to the sudo group.
      url: step-3-grant-sudo-privileges
    - name: Test the new account
      text: Switch to the new user with su - username and test sudo access by running sudo whoami.
      url: step-4-test-the-new-sudo-user
faq:
  - question: "What is the sudo group on Ubuntu/Debian?"
    answer: "The <code>sudo</code> group is a system group that grants its members the permission to run any command with root-level privileges by typing <code>sudo</code> before the command."
  - question: "How does the adduser command differ from useradd on Debian/Ubuntu?"
    answer: "The <code>adduser</code> command is a user-friendly perl script that automatically sets up the home directory, prompts for a password, and copies default configuration files. The <code>useradd</code> command is a low-level utility that does not create these by default unless specified with flags."
  - question: "How do I remove sudo access from a user on Ubuntu?"
    answer: "You can remove a user from the <code>sudo</code> group by running <code>sudo deluser username sudo</code>."
  - question: "How do I delete a user account and their home directory?"
    answer: "You can delete a user and purge their home directory/files by running the command <code>sudo deluser --remove-home username</code>."
  - question: "Why does the system ask for my password when running a sudo command?"
    answer: "This is a security feature to verify that the person at the keyboard is indeed the authorized user, protecting the session in case the terminal was left unattended. Sudo caches the authentication for 15 minutes by default."
status: published
locale: en
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

When you deploy a new server, whether it's a cloud VPS or a bare-metal machine, the hosting provider usually hands you the `root` credentials. The root account has absolute power over the system, no restrictions, no confirmation prompts, no safety net. One wrong command and you can wipe the entire disk.

Logging in directly as root is widely considered a bad security practice. Instead, you should create a regular user account, give it administrative privileges (via `sudo`), and use that account for your day-to-day work.

On Ubuntu and Debian, creating a new sudo user takes about two minutes.

## Step 1: Log in as root

First, connect to your server via SSH as the `root` user:

```bash
ssh root@your-server-ip
```

## Step 2: Add the new user

Use the `adduser` command to create the new account. Replace `yourusername` with whatever name you want to use (don't use spaces or uppercase letters):

{% image "/assets/images/blog/en/add-sudo-user-ubuntu/H1.png", "Running adduser command in Ubuntu terminal to create a new non-root user account", "(max-width: 768px) 100vw, 800px" %}

```bash
adduser yourusername
```

You'll be prompted to set and confirm a new password. Make it strong, if your server has SSH password authentication enabled, bots will eventually try to guess it.

```text
Adding user `yourusername' ...
Adding new group `yourusername' (1001) ...
Adding new user `yourusername' (1001) with group `yourusername' ...
Creating home directory `/home/yourusername' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
```

After the password, you'll be asked for some user information (Full Name, Room Number, etc.). This is a relic from the early days of Unix. You don't need to fill any of it out, just press `ENTER` to skip through all of them, and press `Y` when asked if the information is correct.

## Step 3: Grant sudo privileges

By default, new users on Ubuntu and Debian are standard, unprivileged accounts. They can't install software or edit system configurations. 

To grant administrative powers, you need to add the user to the `sudo` group. Members of this group are automatically allowed to run any command with root privileges by typing `sudo` before it.

Run this command (again, replace `yourusername`):

{% image "/assets/images/blog/en/add-sudo-user-ubuntu/H2.png", "Running usermod -aG sudo command on Ubuntu to grant sudo privileges to a non-root user", "(max-width: 768px) 100vw, 800px" %}

```bash
usermod -aG sudo yourusername
```

The `-aG` flags are important. `-a` means "append" and `-G` means "groups". If you forget the `-a`, the user will be removed from all their other groups and added *only* to the sudo group, which breaks things.

## Step 4: Test the new sudo user

Before you log out of your root session, make sure the new user works. Use the `su` (switch user) command to instantly switch to the new account:

{% image "/assets/images/blog/en/add-sudo-user-ubuntu/H3.png", "Using su command in Ubuntu to switch user accounts and test new user access", "(max-width: 768px) 100vw, 800px" %}

```bash
su - yourusername
```

Your command prompt will change from `root@server` to `yourusername@server`. Now, verify that your sudo privileges work by running a command that requires root access, like checking the root directory or testing the `whoami` command:

{% image "/assets/images/blog/en/add-sudo-user-ubuntu/H4.png", "Running sudo whoami on Ubuntu to verify that sudo privileges were successfully granted to new user", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo whoami
```

The very first time you use `sudo` on a new account, you'll see a standard lecture about respecting others' privacy and thinking before you type. It will then ask for your password (the one you set in Step 1, not the root password).

If the output says `root`, success! Your user has administrative privileges.

## Next Steps

Now that you have a safer way to administer your server, you can close your SSH session and log back in directly as the new user:

```bash
ssh yourusername@your-server-ip
```

For a truly secure server, your next move should be setting up SSH key authentication for this new user and [disabling root login entirely](/blog/secure-ssh-ubuntu-debian/).

If you're looking for a clean, fast environment to host your projects, deploy a [Budget VPS](/budget-vps/), create your user, and start building.