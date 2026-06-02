---
image: /assets/images/blog/en/configure-firewalld-centos-rhel/og-image.png
title: 'How to Configure firewalld on AlmaLinux, CentOS, Rocky Linux & Fedora: The Complete Server Guide'
description: A complete step-by-step guide to setting up firewalld on AlmaLinux, CentOS Stream, Rocky Linux, and Fedora servers. Learn how to manage zones, open ports, list services, and secure your VPS.
date: '2026-03-25'
updated: '2026-06-02'
translationKey: configure-firewalld-rhel-fedora
category: Tutorials
tags:
  - almalinux
  - centos
  - rocky linux
  - fedora
  - firewalld
  - firewall
  - security
  - linux
  - vps
  - server administration
howto:
  name: How to Configure firewalld on AlmaLinux, CentOS Stream, Rocky Linux & Fedora
  totalTime: PT10M
  yield: A secured RHEL-family or Fedora server with a properly configured firewalld setup
  tool:
    - A VPS or dedicated server running AlmaLinux, CentOS Stream, Rocky Linux, or Fedora
    - SSH client (e.g. terminal, PuTTY)
    - A user account with sudo privileges
  steps:
    - name: Start the firewalld service
      text: Run sudo systemctl enable --now firewalld to activate the firewall on your system.
      url: step-1-start-and-enable-firewalld
    - name: Check your active zones
      text: Run sudo firewall-cmd --get-active-zones to see which network interfaces belong to the public zone.
      url: step-2-understanding-zones
    - name: List current firewall rules
      text: Run sudo firewall-cmd --list-all to review currently open ports and services.
      url: step-3-check-your-current-rules
    - name: Open ports and services
      text: Use sudo firewall-cmd --permanent --add-service=http to allow traffic permanently.
      url: step-4-allow-services-and-ports
    - name: Reload the firewall
      text: Run sudo firewall-cmd --reload to apply all permanent rule changes.
      url: step-5-reload-and-verify
faq:
  - question: "What is firewalld and how is it different from UFW?"
    answer: "Firewalld is a dynamic, zone-based firewall manager used primarily in the RHEL ecosystem (AlmaLinux, CentOS, Rocky Linux, Fedora). Unlike UFW which assumes a single connection, firewalld supports multiple security zones to apply different rules based on the network interface."
  - question: "Why is the --permanent flag important in firewall-cmd?"
    answer: "Without the <code>--permanent</code> flag, firewall-cmd rules are applied immediately to the running state but will be lost when the system reboots or the service restarts. Adding <code>--permanent</code> writes the rules to disk, requiring a reload to activate them."
  - question: "How do I temporarily disable or stop firewalld?"
    answer: "You can stop the firewall by running <code>sudo systemctl stop firewalld</code>. To ensure it doesn't start automatically on boot, run <code>sudo systemctl disable firewalld</code>."
  - question: "How do I check which zone is currently the default?"
    answer: "You can find the default zone name by executing the command <code>sudo firewall-cmd --get-default-zone</code>. By default, on fresh installations, this is set to the 'public' zone."
  - question: "What is a Rich Rule in firewalld?"
    answer: "A Rich Rule is an advanced configuration statement in firewalld that allows you to create highly granular rules, such as allowing access to a specific port only from a specific IP address or blocking particular traffic types."
status: published
locale: en
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

If you followed our guide on securing Ubuntu, you probably learned about UFW. But on the Red Hat side of the Linux world, AlmaLinux, CentOS Stream, Rocky Linux, and Fedora, the default, officially supported firewall manager is **firewalld**.

While UFW assumes your server just has one simple connection to the internet, `firewalld` uses a concept called "Zones". This allows you to have completely different rules for your public internet connection vs. your private local network connecting your database servers.

For a standard web server or VPS, though, it's just as easy to set up. Here is everything you need to know to lock down your system.

## Step 1: Start and Enable firewalld

On most RHEL-family distributions, `firewalld` is installed by default but might not be running.

{% image "/assets/images/blog/en/configure-firewalld-centos-rhel/H1.png", "Running sudo systemctl enable --now firewalld on AlmaLinux or Rocky Linux to start and enable the firewalld service on boot", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl enable --now firewalld
sudo systemctl status firewalld
```

If the status is "active (running)", you're good. If it's not installed, you can get it through the `dnf` package manager:

```bash
sudo dnf install firewalld -y
sudo systemctl enable --now firewalld
```

## Step 2: Understanding "Zones"

`firewalld` organizes rules into "zones" based on the trust level of the network interface. For a VPS attached directly to the public internet, all your traffic usually comes through the `public` zone.

Find out which zone your main network interface is in:

{% image "/assets/images/blog/en/configure-firewalld-centos-rhel/H2.png", "Running sudo firewall-cmd --get-active-zones on AlmaLinux to display which firewalld zones are active and which interfaces are assigned to them", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo firewall-cmd --get-active-zones
```

You'll see output like this:
```
public
  interfaces: eth0
```

This confirms your active internet connection (`eth0`) uses the `public` zone. Unless you specify otherwise, all commands you run will apply to this default zone.

## Step 3: Check Your Current Rules

Before blindly adding rules, see what is already open:

{% image "/assets/images/blog/en/configure-firewalld-centos-rhel/H3.png", "Running sudo firewall-cmd --list-all on AlmaLinux to list all currently active firewall rules and open services in the public zone", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo firewall-cmd --list-all
```

RHEL distros generally ship with `ssh` and `dhcpv6-client` enabled out of the box so you don't accidentally lock yourself out immediately. A "deny all incoming" rule is implicitly applied to anything *not* on this list.

## Step 4: Allow Services and Ports

The cleanest way to use firewalld is by allowing predefined **services**. A "service" in firewalld is simply a labeled rule for one or more ports.

For example, to open your server up to HTTP and HTTPS web traffic:

{% image "/assets/images/blog/en/configure-firewalld-centos-rhel/H4.png", "Running sudo firewall-cmd --permanent --add-service=http to permanently allow HTTP traffic through firewalld on AlmaLinux or Rocky Linux", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
```

The `--permanent` flag is extremely important. If you omit it, the rule applies instantly but disappears the next time the server reboots. By adding `--permanent`, the rule gets written to disk, ensuring it survives a system reboot.

### Opening Specific Ports

If you're running a custom application, say, a Node.js app on port 3000, use the port format instead:

```bash
sudo firewall-cmd --permanent --add-port=3000/tcp
```

### Deleting Rules

Added something by accident? The reverse command is identical, just swap "add" for "remove":

```bash
sudo firewall-cmd --permanent --remove-port=3000/tcp
```

## Step 5: Reload and Verify

Because you used the `--permanent` flag in Step 4, none of your new rules are actively running yet. They only exist on the disk. To push the saved rules into the active firewall state, you must reload:

{% image "/assets/images/blog/en/configure-firewalld-centos-rhel/H5.png", "Running sudo firewall-cmd --reload on AlmaLinux to apply permanent firewall rule changes without restarting the server", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo firewall-cmd --reload
```

Finally, run `--list-all` again to verify your new policies are in place:

{% image "/assets/images/blog/en/configure-firewalld-centos-rhel/H6.png", "Running sudo firewall-cmd --list-all after reload on AlmaLinux to confirm the new HTTP and HTTPS rules are now active", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo firewall-cmd --list-all
```

You should see your newly added services and ports under the `services:` and `ports:` line.

## Advanced: Allowing Specific IPs

What if you have a MySQL database on port `3306`, and you only want your application server (`203.0.113.55`) to access it, not the whole internet? 

You can use a "Rich Rule" to tie a port to a specific source IP permanently:

```bash
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="203.0.113.55" port protocol="tcp" port="3306" accept'
sudo firewall-cmd --reload
```

## Wrapping Up

Once configured, `firewalld` runs quietly in the background securing the system. Since it natively integrates into the core of Fedora, Rocky, CentOS and AlmaLinux, it gracefully handles complex operations like bridging containers or routing internal networks natively. Coupled with [fail2ban](/blog/setup-fail2ban-centos-rhel/), your server is highly resistant to generic brute-force probing from internet bots.

To test your security skills on the live internet without risking any production payloads, checking out a temporary, low-tier [Budget VPS](/budget-vps/) is a cheap, reliable way to practice configuring an unbreachable RHEL system.