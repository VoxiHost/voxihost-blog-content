---
image: /assets/images/blog/en/update-ubuntu-debian/og-image.png
title: 'How to Update Ubuntu & Debian: The Complete Server Guide'
description: A complete step-by-step guide on how to update Ubuntu and Debian Linux servers. Covers apt update, apt upgrade, kernel updates, automatic updates, and best practices for production VPS environments.
date: '2026-03-24'
updated: '2026-06-02'
translationKey: update-ubuntu-debian
category: Tutorials
tags:
  - ubuntu
  - debian
  - apt update
  - apt upgrade
  - linux
  - vps
  - server administration
  - unattended-upgrades
  - kernel update
howto:
  name: How to Update Ubuntu & Debian Linux Server
  totalTime: PT5M
  yield: A fully updated Ubuntu or Debian server with the latest security patches applied
  tool:
    - A VPS or dedicated server running Ubuntu or Debian
    - SSH client (e.g. terminal, PuTTY)
    - sudo or root access
  steps:
    - name: Refresh the package index
      text: Run sudo apt update to sync the local package index with the repositories. This does not install anything.
      url: refresh-the-package-index
    - name: Install available updates
      text: Run sudo apt upgrade -y to install all pending updates for already-installed packages.
      url: install-available-updates
    - name: Handle held-back packages
      text: If apt upgrade reports packages kept back, run sudo apt full-upgrade -y to resolve dependency changes.
      url: handling-kept-back-packages-full-upgrade
    - name: Remove orphaned packages
      text: Run sudo apt autoremove -y to clean up old libraries and dependencies no longer needed.
      url: cleaning-up-old-packages-autoremove
    - name: Check if a reboot is required
      text: Run cat /var/run/reboot-required to check if a kernel update requires a system restart.
      url: do-you-need-a-reboot-reboot-required
    - name: Enable automatic security updates
      text: Install and configure unattended-upgrades to automatically apply security patches.
      url: automating-patches-with-unattended-upgrades
faq:
  - question: "What is the difference between apt update and apt upgrade?"
    answer: "The <code>apt update</code> command refreshes the local package list and metadata from repositories (without installing anything), whereas <code>apt upgrade</code> downloads and installs the actual package updates."
  - question: "How do I fix packages kept back on Ubuntu?"
    answer: "Packages are held back when an upgrade requires installing new dependencies or removing old ones. You can safely force the installation of these packages by running <code>sudo apt full-upgrade</code>."
  - question: "Is it safe to run apt upgrade on a production server?"
    answer: "Yes, standard package upgrades are safe, but you should always take a VM snapshot first, run updates during low-traffic hours, and check if a reboot is needed afterwards using the <code>/var/run/reboot-required</code> file."
  - question: "What does apt autoremove do?"
    answer: "The <code>apt autoremove</code> command removes packages (mostly libraries or old kernel versions) that were automatically installed to satisfy dependencies for other packages but are no longer needed by any installed software."
  - question: "How do I automate security updates on Ubuntu and Debian?"
    answer: "You can enable automatic background updates by installing the <code>unattended-upgrades</code> package and configuring the package manager behavior in <code>/etc/apt/apt.conf.d/50unattended-upgrades</code>."
status: published
locale: en
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

If you've spun up a fresh VPS and aren't sure what to do first, update it. Sounds obvious, but a surprising number of servers sitting on the public internet are running packages that haven't been touched since the OS was installed. That's a problem.

Ubuntu and Debian ship solid defaults, but "solid" doesn't mean "secure forever". Packages get CVEs every week. The kernel gets patched. OpenSSH, OpenSSL, curl, all of them have had serious vulnerabilities over the years that were already fixed in an update most people hadn't bothered to apply. So let's fix that.

Before we start: if you are deploying a fresh server with a premium provider like **<span class="text-white">Voxi</span><span class="text-amber-300">Host</span>**, the system automatically runs a full package update immediately after deployment on first boot. But as your server runs over time, you will still need to know how to maintain it yourself.

## Refresh the package index

Before anything else, refresh your local package index. This doesn't install anything, it just checks what updates are actually out there:

{% image "/assets/images/blog/en/update-ubuntu-debian/H1.png", "Running sudo apt update on Ubuntu or Debian to refresh the package index and check for available updates", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt update
```

## Install available updates

Then install them:

{% image "/assets/images/blog/en/update-ubuntu-debian/H2.png", "Running sudo apt upgrade -y on Ubuntu or Debian to install all available package upgrades from the updated index", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt upgrade -y
```

That's it for routine maintenance. Run these two, you're done. The `-y` flag skips the confirmation prompt, which is handy when you're running this in a script or just don't want to babysit the terminal.

One thing worth knowing: `apt upgrade` won't remove packages or pull in new dependencies. It only updates what's already installed. That's intentional. It makes it safe to run on a live server without worrying about something breaking because a package got swapped out.

## Handling "kept back" packages (full-upgrade)

Sometimes after running `apt upgrade` you'll see a line saying a few packages were kept back. That usually means those packages have updated dependencies that aren't installed yet, and `apt upgrade` is playing it safe.

To deal with those, switch to `full-upgrade`:

```bash
sudo apt full-upgrade -y
```

This can pull in new dependencies or remove conflicting packages. It's not dangerous, but it does more than a regular upgrade, so it's worth checking what it plans to do before confirming on a critical machine.

## Cleaning up old packages (autoremove)

Upgrades tend to leave orphaned packages behind: old libraries that were dependencies of something that got updated. Clean those up with:

{% image "/assets/images/blog/en/update-ubuntu-debian/H3.png", "Running sudo apt autoremove -y on Ubuntu or Debian to remove old dependency packages no longer needed after upgrades", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt autoremove -y
```

Worth running after any major upgrade. You're not losing anything important, just clearing dead weight.

## Do you need a reboot? (reboot-required)

Kernel updates don't take effect until you restart the machine. Ubuntu and Debian leave a breadcrumb when a reboot is pending:

{% image "/assets/images/blog/en/update-ubuntu-debian/H4.png", "Checking /var/run/reboot-required file on Ubuntu to determine if a system reboot is needed after a kernel update", "(max-width: 768px) 100vw, 800px" %}

```bash
cat /var/run/reboot-required
```

If that file exists, you need to reboot. If you want to know specifically which packages are causing it:

```bash
cat /var/run/reboot-required.pkgs
```

On production servers, pick your window. Rebooting mid-afternoon on a busy game server is going to upset people. If you're on a VPS with a management panel, most providers let you schedule restarts or at least see current traffic before pulling the trigger.

## Automating patches with unattended-upgrades

For servers you don't log into every week, automatic security updates are a reasonable safety net. The `unattended-upgrades` package handles this:

```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

By default it only applies security patches (not regular package upgrades) which is exactly what you want. You're not automatically getting new feature versions of things, just staying patched against known vulnerabilities.

## The quick update one-liner

When you SSH in to do something quick and want to make sure the system is current before you leave:

```bash
sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y
```

Chain it, let it run, check if a reboot is needed, done.

## Upgrading to a new release (do-release-upgrade)

If you want to jump from Ubuntu 22.04 to 24.04:

{% image "/assets/images/blog/en/update-ubuntu-debian/H5.png", "Running sudo do-release-upgrade on Ubuntu to upgrade from one major release to the next, for example 22.04 to 24.04", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo do-release-upgrade
```

Take a snapshot first. This is not a small operation, as it touches essentially everything on the system. If you're on a production server, seriously consider doing this on a clone first and testing your applications on the new version before committing.

## What to actually watch out for

Kernel and libc updates are the ones that always need a reboot. Config file changes are the ones that can trip you up. During upgrades, if a package ships a new default config for something you've customized, apt will ask whether to keep your version or install the new one. Read those prompts carefully.

Regularly updated servers also fail in more predictable ways. If something breaks after an update, you know exactly when it happened and what packages changed. On a server that hasn't been updated in 8 months, debugging a failure is a much messier experience.

For a completely hardened server, package updates are just step one. Consider setting up [UFW Firewall](/blog/configure-ufw-ubuntu-debian/) and [fail2ban](/blog/setup-fail2ban-ubuntu-debian/) to drop malicious background noise actively.

If you want a clean VPS to practice this on, our [Budget VPS](/budget-vps/) plans are cheap enough that you can snapshot, experiment, and blow it up without any stress.