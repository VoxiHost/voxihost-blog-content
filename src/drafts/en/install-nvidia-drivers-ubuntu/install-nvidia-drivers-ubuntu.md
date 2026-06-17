---
image: /assets/images/blog/en/install-nvidia-drivers-ubuntu/og-image.png
title: 'Complete Guide: Installing NVIDIA Drivers on Ubuntu and AnduinOS'
description: Step-by-step instructions for installing proprietary NVIDIA drivers on Ubuntu-based systems, including Secure Boot signing and manual installation methods.
date: '2026-06-17'
translationKey: install-nvidia-drivers-ubuntu
locale: en
category: Tutorials
tags:
  - nvidia
  - drivers
  - ubuntu
  - linux
  - gpu
status: draft
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - your-github-username
howto:
  name: Install NVIDIA Drivers on Ubuntu
  totalTime: PT25M
  yield: A system with fully functional proprietary NVIDIA drivers and GPU acceleration enabled.
  tool:
    - A computer or VPS with an NVIDIA GPU
    - Ubuntu or an Ubuntu-based distribution (like AnduinOS)
    - Internet connection
  steps:
    - name: Step 1 — Automatic Installation
      text: Use the built-in ubuntu-drivers tool to install the recommended version.
      url: step-1--automatic-installation
    - name: Step 2 — (Optional) PPA Installation
      text: Add the graphics-drivers PPA to get the latest stable versions.
      url: step-2--optional-ppa-installation
    - name: Step 3 — Manual Installation with Secure Boot
      text: Sign the kernel module if Secure Boot is enabled on your system.
      url: step-3--manual-installation-with-secure-boot
---

## Introduction

Getting the best performance out of your NVIDIA GPU on Linux requires the proprietary drivers. While the open-source Nouveau driver is useful for basic display tasks, it lacks the optimizations needed for gaming, video editing, and AI workloads on your <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> workstation or server.

In this guide, we will cover the three main ways to install NVIDIA drivers on Ubuntu-based systems, ranging from the safest automatic method to the most advanced manual installation.

{% image "/assets/images/blog/en/install-nvidia-drivers-ubuntu/hero.png", "NVIDIA logo with a Linux terminal background", "(max-width: 768px) 100vw, 800px" %}

> **Prerequisites:** Ensure your system is up to date and you have a compatible NVIDIA GPU. Always [backup your system](/premium-vps/) before making major driver changes.

---

## Step 1 — Automatic Installation

The simplest way to install drivers is to let the system detect your hardware and choose the best version for you.

```bash
sudo apt update
sudo ubuntu-drivers install
```

After the installation finishes, simply **reboot** your system. This method is highly recommended for most users as it handles dependencies and DKMS automatically.

---

## Step 2 — (Optional) PPA Installation

If you need a newer driver than what is available in the default repositories (e.g., for a brand new RTX 40-series card), use the official Graphics Drivers PPA.

```bash
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update
```

Now you can list and install specific versions:

```bash
ubuntu-drivers list
sudo apt install nvidia-driver-550 # Replace with the desired version
```

---

## Step 3 — Manual Installation with Secure Boot

If you are using the `.run` installer from NVIDIA's website and have **Secure Boot** enabled, you must sign the kernel module.

### Generate a Signing Key

```bash
mkdir ~/mok-keys && cd ~/mok-keys
openssl req -new -newkey rsa:2048 -days 36500 -nodes -keyout MOK.key -out MOK.csr
openssl x509 -req -in MOK.csr -signkey MOK.key -out MOK.crt
openssl x509 -in MOK.crt -outform DER -out MOK.der
```

### Enroll the Key

```bash
sudo mokutil --import MOK.der
```

Set a password, then **reboot**. During boot, select "Enroll MOK" in the blue MokManager screen and enter your password.

### Run the Installer

Once back in Ubuntu, stop the display manager and run the installer, providing the path to your keys when prompted:

```bash
sudo ./NVIDIA-Linux-x86_64-xxx.xx.run --module-signing-secret-key=/home/your-user/mok-keys/MOK.key --module-signing-public-key=/home/your-user/mok-keys/MOK.crt
```

---

## Conclusion

Whether you chose the automatic method or the manual one, you should now have fully functional NVIDIA drivers. You can verify the installation by running:

```bash
nvidia-smi
```

If you see a table with your GPU details, you are ready to go! For those running GPU-accelerated containers, don't forget to check out the [NVIDIA Container Toolkit](/blog/docker-nvidia-container-toolkit/).

Looking for a high-performance environment for your AI projects? Check out <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Premium VPS plans](/premium-vps/) with dedicated resources.
