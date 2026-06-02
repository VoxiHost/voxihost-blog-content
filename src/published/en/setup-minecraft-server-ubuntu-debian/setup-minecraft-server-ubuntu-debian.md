---
image: /assets/images/blog/en/setup-minecraft-server-ubuntu-debian/og-image.png
title: How to Set Up a Minecraft Java Edition Server on Ubuntu/Debian
description: A complete guide to hosting a Minecraft Java Edition server on Linux. Learn how to install and configure the correct Java environment for every version.
date: '2026-04-22'
updated: '2026-06-02'
translationKey: minecraft-vanilla-server-setup-ubuntu-debian
locale: en
category: Tutorials
tags:
  - minecraft
  - java edition
  - vps
  - linux
  - ubuntu
  - debian
  - server setup
faq:
  - question: "Why does Java version matter for a Minecraft server?"
    answer: "Minecraft's game engine has evolved over time, and different versions require different Java environments to run. Using the wrong version of Java (e.g. Java 8 for Minecraft 1.21) is the most common cause of startup crashes."
  - question: "What is the difference between openjdk and openjdk-headless?"
    answer: "The <code>-headless</code> package excludes graphical libraries (GUI), which are not needed on a terminal-only server. Installing the headless version saves disk space and RAM."
  - question: "Can I install multiple versions of Java on the same server?"
    answer: "Yes, you can install multiple Java versions side-by-side. You can switch the default system version using <code>sudo update-alternatives --config java</code>, or specify the exact path to the Java binary in your server startup script."
  - question: "Is this guide compatible with Bedrock Edition?"
    answer: "No. This guide is specifically for Minecraft Java Edition. Bedrock Edition (consoles, mobile, and Windows 10/11) uses a completely different C++ server engine that does not require Java."
  - question: "How do I upgrade from Vanilla to Paper or Spigot?"
    answer: "Upgrading is simple: replace the Vanilla <code>server.jar</code> file with the <code>paper.jar</code> or <code>spigot.jar</code> file, and update your startup script to point to the new file. Your world files will automatically be converted."
status: published
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

> **Edition Choice:** This guide is specifically for **Minecraft Java Edition**. It is not compatible with Bedrock Edition (consoles, mobile, or Windows 10 "Bedrock" app).

Setting up your own Minecraft Java Edition server gives you complete control over your world and community. At **<span class="text-white">Voxi</span><span class="text-amber-300">Host</span>**, our [Budget VPS](/budget-vps/) and [Premium VPS](/premium-vps/) environments provide the perfect, dedicated resources needed to run a smooth, lag-free server on Linux distributions like Ubuntu or Debian.

However, the most common hurdle for new server administrators isn't the Linux command line. It's **Java** itself.

Minecraft Java Edition is built on Java. Over the years, the game's engine has evolved, requiring newer, more advanced versions of the Java environment to run. Using the wrong version of Java is the **#1 cause of server crash errors** right at startup.


## The Golden Rule: Match Your Java to Your Minecraft

Before you type a single command into your server's terminal, you must know which version of Minecraft you intend to host. The official Vanilla server software (provided by Mojang) will simply refuse to start if the installed Java Development Kit (JDK) or Java Runtime Environment (JRE) doesn't match its requirements.

Here is the definitive breakdown of Minecraft versions and their corresponding Java requirements:

| Minecraft Version | Java Requirements | Recommended Package | Importance |
| :--- | :--- | :--- | :--- |
| [1.20.5 – 1.21.x](/blog/minecraft-1-21-server-ubuntu-debian/) | **Java 21** | `openjdk-21-jre-headless` | The modern standard. Essential for the latest Vanilla features. |
| [1.18 – 1.20.4](/blog/minecraft-1-19-server-ubuntu-debian/) | **Java 17** | `openjdk-17-jre-headless` | The backbone of many active survival worlds. |
| [1.17 – 1.17.1](/blog/minecraft-1-17-server-ubuntu-debian/) | **Java 16** | `openjdk-16-jre-headless` | A transitional update. Most admins skip this for 1.18+. |
| [1.7.10 – 1.16.5](/blog/minecraft-1-8-8-server-ubuntu-debian/) | **Java 8** | `openjdk-8-jre-headless` | The classic era. Extreme stability for legacy and PvP. |

> **Pro Tip for VPS Users:** Notice that we always recommend the `-headless` packages (e.g., `openjdk-21-jre-headless`). Headless packages exclude graphical user interface (GUI) libraries that are useless on a terminal-only server. This saves disk space and preserves your RAM for the Minecraft server itself.

## Why Start with Vanilla?

While there are many custom server engines out there (like Paper or Forge), starting with the official **Vanilla server software** from Mojang is the best way to understand how Minecraft hosting fundamentally works.

* **100% Compatibility**: Guarantees all official game mechanics work exactly as intended.
* **No Modifications**: No third-party code interfering with mob spawning or redstone.
* **Easy Upgrade Path**: Once you master Vanilla, moving to Paper or Spigot is incredibly simple.

## Step-by-Step Installation Guides

Because the setup commands differ significantly depending on the "Java Era" you choose, we have broken down the actual installation into specific, easy-to-follow tutorials:

### 1. Modern Vanilla (1.20.5+ / Java 21)
Ready for the newest updates and features straight from Mojang? This guide covers setting up the latest Vanilla server using the Java 21 environment.
👉 **[Setting Up a Vanilla 1.20.5+ Server (Java 21)](/blog/minecraft-1-21-server-ubuntu-debian/)**

### 2. The Great Update Era (1.18 - 1.20.4 / Java 17)
Hosting a world in the 1.18+ era? This setup focuses on Java 17, which brought major performance improvements and engine changes.
👉 **[Setting Up a Vanilla 1.18 - 1.20.4 Server](/blog/minecraft-1-19-server-ubuntu-debian/)**

### 3. Transitional Versions (1.17 / Java 16)
If you specifically need to run 1.17.x, you'll need the short-lived Java 16 environment.
👉 **[Setting Up a Vanilla 1.17 Server](/blog/minecraft-1-17-server-ubuntu-debian/)**

### 4. Classic Vanilla (1.7.10 - 1.16.5 / Java 8)
Looking to experience the game exactly as it was in the classic era? Deploy a pure Vanilla environment on Java 8 for maximum compatibility.
👉 **[Setting Up a Classic 1.8.8 Vanilla Server](/blog/minecraft-1-8-8-server-ubuntu-debian/)**

## Conclusion

Understanding the connection between Minecraft versions and Java versions is the most important step toward successful server administration. Whether you lean toward our [Budget VPS](/budget-vps/) for a cost-effective start or our [Premium VPS](/premium-vps/) for a high-performance community world, ensuring the correct Java environment is the key to a stable launch. 

Pick your target version from the requirements table above and follow our specific setup guides to bring your Minecraft Java Edition world online!