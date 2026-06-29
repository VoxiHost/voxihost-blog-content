---
image: /assets/images/blog/en/install-nvidia-drivers-ubuntu/og-image.png
title: 'Complete Guide: Installing NVIDIA Drivers on Ubuntu and AnduinOS'
description: Install proprietary NVIDIA drivers on Ubuntu-based systems, including Secure Boot module signing and manual installer methods.
date: '2026-06-29'
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
  name: Anduin
  link: https://github.com/Anduin2017
contributors:
  - Anduin2017
  - danielmarszalkowski
howto:
  name: Install NVIDIA Drivers on Ubuntu
  totalTime: PT25M
  yield: A system with fully functional proprietary NVIDIA drivers and GPU acceleration enabled.
  tool:
    - A computer or VPS with an NVIDIA GPU
    - Ubuntu or an Ubuntu-based distribution (like AnduinOS)
    - Internet connection
  steps:
    - name: "Step 1: Automatic Installation"
      text: Use the built-in ubuntu-drivers tool to install the recommended version.
      url: step-1-automatic-installation
    - name: "Step 2: (Optional) PPA Installation"
      text: Add the graphics-drivers PPA to get the latest stable versions.
      url: step-2-optional-ppa-installation
    - name: "Step 3: Manual Installation with Secure Boot"
      text: Sign the kernel module if Secure Boot is enabled on your system.
      url: step-3-manual-installation-with-secure-boot
faq:
  - question: "Why does nvidia-smi fail after installing the driver?"
    answer: "This is usually because the system hasn't been rebooted yet to load the new kernel modules, or because UEFI Secure Boot is active and has blocked the unsigned NVIDIA driver from loading."
  - question: "How do I switch back to the open-source Nouveau driver?"
    answer: "You can restore the default Nouveau drivers by purging the proprietary NVIDIA packages: run <code>sudo apt purge nvidia-*</code> and then install the Nouveau driver package with <code>sudo apt install xserver-xorg-video-nouveau</code>."
  - question: "Can I use nvidia-smi on any VPS instance?"
    answer: "No, NVIDIA GPU tools require a physical graphics card attached to the machine. You can only use them if your VPS plan supports dedicated GPU passthrough or you use dedicated GPU bare-metal servers."
---

## Introduction

For optimal performance from an NVIDIA GPU on Linux, you must use the proprietary drivers. Although the open-source Nouveau driver suffices for basic display tasks, it lacks the optimizations necessary for gaming, GPU acceleration, and AI workloads on your <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> workstation or server.

This guide covers the three main installation methods for NVIDIA drivers on Ubuntu-based systems, starting with the recommended automated tools and ending with a detailed manual installation procedure.

> **Prerequisites:** Ensure your system is up to date and you have a compatible NVIDIA GPU. Always [backup your system](/premium-vps/) before making major driver changes.

---

## Step 1: Automatic Installation

The recommended approach is to let the system detect the hardware automatically and install the appropriate driver version:

```bash
sudo apt update
sudo ubuntu-drivers install
```

Reboot your system once the installation completes. This method is ideal for most setups since it manages system updates and DKMS (Dynamic Kernel Module Support) automatically.

{% image "/assets/images/blog/en/install-nvidia-drivers-ubuntu/H1.png", "Terminal showing system hardware detection and recommended proprietary NVIDIA drivers", "(max-width: 768px) 100vw, 800px" %}

---

## Step 2: (Optional) PPA Installation

If your hardware requires a newer driver version than the default repositories provide (such as for recent graphics card architectures), use the official graphics drivers PPA:

```bash
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update
```

You can then query the available versions and install the required one:

```bash
ubuntu-drivers list
sudo apt install nvidia-driver-550 # Replace with the desired version
```

---

## Step 3: Manual Installation with Secure Boot

If you prefer the official manual `.run` installer and have UEFI Secure Boot enabled, you must sign the kernel module before the system will load it.

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

Set a secure one-time password and reboot. During system boot, the blue MokManager utility will appear: choose "Enroll MOK", confirm, and enter the password.

### Run the Installer

After boot, stop your display manager (if active) and launch the installer, passing the key paths as arguments:

```bash
sudo ./NVIDIA-Linux-x86_64-xxx.xx.run --module-signing-secret-key=/home/your-user/mok-keys/MOK.key --module-signing-public-key=/home/your-user/mok-keys/MOK.crt
```

---

## Conclusion

Once the installation completes, verify the GPU status by running:

```bash
nvidia-smi
```

{% image "/assets/images/blog/en/install-nvidia-drivers-ubuntu/H2.png", "Terminal display of the nvidia-smi tool showing GPU details and driver version", "(max-width: 768px) 100vw, 800px" %}

If you see a table with your GPU details, you are ready to go! For those running GPU-accelerated containers, don't forget to check out the [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html).

Looking for a high-performance environment for your AI projects? Check out <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Premium VPS plans](/premium-vps/) with dedicated resources.
