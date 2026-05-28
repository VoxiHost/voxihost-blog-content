---
image: /assets/images/blog/update-fedora/og-image.png
title: 'How to Update Fedora 43 & Newer: The Complete Server Guide'
description: A complete step-by-step guide to updating Fedora 43 and newer servers using dnf5. Covers dnf5 upgrade, autoremove, reboot detection, and automatic updates for production VPS environments.
date: '2026-03-25'
translationKey: update-fedora
category: Tutorials
tags:
  - fedora
  - dnf5
  - dnf upgrade
  - linux
  - vps
  - server administration
  - dnf-automatic
  - fedora 43
howto:
  name: How to Update Fedora 43 Server
  totalTime: PT5M
  yield: A fully updated Fedora 43 server with the latest security patches applied
  tool:
    - A VPS or dedicated server running Fedora 43
    - SSH client (e.g. terminal, PuTTY)
    - sudo or root access
  steps:
    - name: Check for available updates
      text: Run sudo dnf5 check-upgrade to see what updates are available without installing anything.
      url: the-basics-dnf5-upgrade
    - name: Install all available updates
      text: Run sudo dnf5 upgrade -y to download and install all pending package updates.
      url: the-basics-dnf5-upgrade
    - name: Remove orphaned packages
      text: Run sudo dnf5 autoremove -y to clean up old libraries no longer needed by any package.
      url: cleaning-up-dnf5-autoremove
    - name: Check if a reboot is required
      text: Run sudo needs-restarting -r to verify whether a kernel update requires a full system restart.
      url: do-you-need-a-reboot-needs-restarting
    - name: Enable automatic security updates
      text: Install and configure dnf-automatic to automatically apply security patches on a schedule.
      url: automating-patches-with-dnf-automatic
status: published
locale: en
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Fedora moves fast. It's the distro that ships what RHEL will be running in two years, which means you get bleeding-edge kernels, newer toolchains, and packages that are actually current. The tradeoff is you need to stay on top of updates more actively than you would on a long-term-support distro.

Starting with **Fedora 41**, the default package manager switched to **dnf5**, a full rewrite of `dnf` that's faster, uses less memory, and has a cleaner API. If you're on Fedora 43, you're using dnf5. The command is `dnf5`, though `dnf` still works as an alias pointing to the same binary.

Before we start: if you are deploying a fresh server with a premium provider like **<span class="text-white">Voxi</span><span class="text-amber-300">Host</span>**, the system automatically runs a full package update immediately after deployment on first boot. But as your server runs over time, you will still need to know how to maintain it yourself.

## The basics: dnf5 upgrade

To update a Fedora 43 server, you run:

{% image "/assets/images/blog/update-fedora/H1.png", "Running sudo dnf5 upgrade on Fedora 43 - terminal output showing packages being updated", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo dnf5 upgrade -y
```

This checks for available updates, downloads them, and installs them in one pass. The **`-y` flag skips the confirmation**. That's it for routine updates.

If you want to see what would change before committing:

{% image "/assets/images/blog/update-fedora/H2.png", "Running dnf5 check-upgrade on Fedora to preview available package updates without installing", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo dnf5 check-upgrade
```

Short list, nothing gets touched. **Good habit** before running updates on a machine that's actually serving traffic.

One difference from older `dnf`: `dnf5` separates the concepts of `update` and `upgrade` more clearly. In practice, `dnf5 upgrade` is the command you want, it handles both package updates and dependency resolution. `dnf5 update` is an alias and works the same way.

## Cleaning up (dnf5 autoremove)

After upgrades, orphaned packages pile up. Old kernels, old libraries that newer versions replaced. Clean those up:

```bash
sudo dnf5 autoremove -y
```

Fedora by default also keeps the last two kernel versions installed, which is sensible, it gives you a fallback if something goes sideways. If you want to prune old kernels specifically:

```bash
sudo dnf5 repoquery --installonly --latest-limit=-2 -q
```

That shows you what would be removed. Add `--remove` to actually do it. Be careful on systems where you don't have out-of-band console access.

## Do you need a reboot? (needs-restarting)

Fedora doesn't write a `reboot-required` file like Ubuntu does. Instead, use:

{% image "/assets/images/blog/update-fedora/H3.png", "Running sudo needs-restarting -r on Fedora to check if a reboot is needed after kernel update", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo needs-restarting -r
```

**Exit code 1** means a reboot is needed (usually a kernel update). **Exit code 0** means you're fine. If the tool isn't installed:

{% image "/assets/images/blog/update-fedora/H4.png", "Installing dnf-utils on Fedora with sudo dnf5 install dnf-utils to get needs-restarting", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo dnf5 install dnf-utils -y
```

On a server, you can also check which services are running against outdated libraries and restart only those, avoiding a full system reboot:

```bash
sudo needs-restarting -s
```

This is particularly useful on Fedora, where kernel updates are frequent. Restarting a handful of services is much less disruptive than taking the machine down.

## Automating patches with dnf-automatic

For servers that sit in the background without regular manual attention:

{% image "/assets/images/blog/update-fedora/H5.png", "Installing dnf-automatic package on Fedora for automatic unattended security updates", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo dnf5 install dnf-automatic -y
```

The config file controls what gets updated automatically:

{% image "/assets/images/blog/update-fedora/H6.png", "Editing /etc/dnf/automatic.conf on Fedora to configure unattended updates with nano", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo nano /etc/dnf/automatic.conf
```

If missing package `nano` install it first:

```bash
sudo dnf5 install nano -y
```

Key settings:

```ini
[commands]
# security = only security patches (recommended for servers)
upgrade_type = security

# Actually install the updates, not just download them
apply_updates = yes

# Emit output to the journal
emit_via = stdio
```

Enable the systemd timer to run it daily:

```bash
sudo systemctl enable --now dnf-automatic.timer
```

Verify it's scheduled:

```bash
sudo systemctl status dnf-automatic.timer
```

On Fedora, security patches come through regularly given how active the project is, so automatic security updates make a real difference.

## The quick one-liner

SSH in, do your thing, leave it clean:

```bash
sudo dnf5 upgrade -y && sudo dnf5 autoremove -y
```

Then check `needs-restarting -r`. Two minutes of work, system stays current.

## Upgrading to the next Fedora release

Fedora moves to a new release every six months, and each release is supported for about 13 months. When it's time to move from 43 to 44:

{% image "/assets/images/blog/update-fedora/H7.png", "Running sudo dnf5 system-upgrade download --releasever=43 to start Fedora version upgrade", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo dnf5 system-upgrade download --releasever=44
sudo dnf5 system-upgrade reboot
```

The upgrade happens on the next boot in a controlled environment. This is the **supported path**, don't try to manually swap repos and run `dnf5 distro-sync`, you'll end up in a broken state.

Before upgrading: **snapshot the VM**, read the Fedora 44 release notes for anything that might break your workload, and if possible test on a clone first.

## The SELinux and config file traps

Two things to watch after updates on Fedora:

**SELinux denials.** If a service stops working after an update, check the audit log:

```bash
sudo ausearch -m avc -ts recent
```

If missing package `ausearch` install it first:

```bash
sudo dnf5 install ausearch -y
```

A policy update might have changed what your service is allowed to do, or an updated binary might need a new label. Most of the time this sorts itself out, but it's the first thing to check.

**RPM config file conflicts.** When a package ships a new default config, it creates a **`.rpmnew`** file instead of overwriting yours:

```bash
sudo find /etc -name "*.rpmnew" -o -name "*.rpmsave" 2>/dev/null
```

Check those after any significant update. Merge what matters, delete what doesn't.

Fedora is a solid server operating system if you're staying current. The six-month release cycle sounds fast, but the system-upgrade path is smooth and rarely causes surprises. If you want a clean VPS to test a Fedora upgrade without risking your production machine, our [Budget VPS](/budget-vps/) plans are a cheap way to run through the whole process.