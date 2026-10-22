---
image: /assets/images/blog/en/private-cloud-filebrowser-docker/og-image.png
title: "Install FileBrowser with Docker on Your VPS"
description: "Learn how to install FileBrowser via Docker in 5 minutes and turn your VPS server into a private cloud drive accessible from your web browser."
date: 2026-07-13
translationKey: "private-cloud-filebrowser-docker"
locale: en
category: Tutorials
tags: ["docker", "filebrowser", "cloud", "vps", "linux", "storage"]
status: published
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - sl0ikkk
howto:
  name: "Installing FileBrowser in Docker"
  totalTime: "PT5M"
  yield: "A fully functional private cloud drive in your browser"
  tool:
    - "Any Linux VPS (e.g., from VoxiHost)"
    - "Installed Docker"
  steps:
    - name: "Step 1"
      text: "Pull and run the container"
      url: "step-1-quick-launch-via-docker"
    - name: "Step 2"
      text: "First login"
      url: "step-2-first-login-to-your-drive"
    - name: "Step 3"
      text: "Account configuration"
      url: "step-3-changing-the-default-password-important"
    - name: "Step 4"
      text: "Managing files"
      url: "step-4-uploading-and-sharing-files"
faq:
  - question: "Do I need a domain name to use FileBrowser?"
    answer: "No, FileBrowser works directly via your server's IP address and a port number (e.g. <code>http://YOUR_IP:8080</code>). A domain is optional."
  - question: "Is it safe to expose FileBrowser on a public IP?"
    answer: "After following this guide you will have a strong password set. For extra security, consider placing FileBrowser behind a reverse proxy with HTTPS (e.g. Nginx + Let's Encrypt) or restricting access by IP using a firewall."
  - question: "How do I restart FileBrowser if the server reboots?"
    answer: "The <code>--restart always</code> flag in the Docker run command ensures FileBrowser starts automatically every time the server boots."
  - question: "Can I limit FileBrowser to a specific folder instead of the whole filesystem?"
    answer: "Yes. Replace <code>-v /:/srv</code> with a specific path, for example <code>-v /home/user/files:/srv</code>, to restrict access to that directory only."
---

## Introduction

Owning a VPS server is great, but transferring files to it via the terminal (SFTP) can be cumbersome. How about turning your cheap server into a convenient cloud drive (like Google Drive or Dropbox) that you can operate 100% from your web browser?

Enter **FileBrowser** – an incredibly lightweight file manager. It works blazingly fast, doesn't require a domain name (you can just use your IP address), and uses so few resources that it runs perfectly on even the cheapest [Budget VPS](/budget-vps/) from our lineup!

---

## Step 1: Quick Launch via Docker

Log into your server via terminal. Running FileBrowser comes down to one simple command. Paste the code below and press ENTER:

{% image "/assets/images/blog/en/private-cloud-filebrowser-docker/H1.png", "Terminal output while starting the FileBrowser container", "(max-width: 768px) 100vw, 800px" %}

```bash
docker run -d --name filebrowser -v /:/srv -p 8080:80 --restart always filebrowser/filebrowser
```

**What exactly does this command do?**
- `-p 8080:80` exposes the dashboard on port 8080.
- `-v /:/srv` gives the app access to your server's entire root file system, allowing you to browse and edit absolutely any system file through your browser!

---

## Step 2: First Login to your Drive

That's it for the terminal! Now open your web browser and enter your server's IP address along with port 8080:

`http://SERVER_IP_ADDRESS:8080`

You will see a clean, minimalist login screen.

{% image "/assets/images/blog/en/private-cloud-filebrowser-docker/H2.png", "FileBrowser login screen in the browser", "(max-width: 768px) 100vw, 800px" %}

To log in, use **`admin`** as the username. In modern versions of FileBrowser, there is no static default password for security reasons.
Your **one-time temporary password** was auto-generated. To find it, go back to your terminal and run:

```bash
docker logs filebrowser
```

Look for a line mentioning the randomly generated password in the logs. Copy it and use it to log in.

---

## Step 3: Changing the Default Password (Important!)

Right after logging in, you must secure your drive so no one else can access your files. Click on the **Settings** tab in the left-hand menu, and then go to the **Profile Settings** section.

{% image "/assets/images/blog/en/private-cloud-filebrowser-docker/H3.png", "Password change section in profile settings", "(max-width: 768px) 100vw, 800px" %}

Enter a new, strong password, confirm it, and click the "Update" button. Your private drive is now fully secure.

---

## Step 4: Uploading and Sharing Files

You can now do whatever you want with your drive. In the top right corner, you'll find buttons to create new folders, upload files, and even create empty text files that you can edit directly in the browser!

By selecting any file, you can click the Share icon to generate a special link. You can send this link to a friend so they can download the file directly from your server without needing an account.

{% image "/assets/images/blog/en/private-cloud-filebrowser-docker/H4.png", "FileBrowser interface showing the file sharing feature", "(max-width: 768px) 100vw, 800px" %}

---

## Conclusion

FileBrowser is a "must-have" for any private server. It uses a fraction of a percent of RAM and offers massive convenience for managing files in Linux. Use this solution on your new [Premium VPS](/premium-vps/) and forget about using clumsy FTP programs!